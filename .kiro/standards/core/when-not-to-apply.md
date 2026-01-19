# When NOT to Apply Patterns - Anti-Pattern Guide

## Purpose

This document explicitly defines when NOT to apply coding patterns and standards. Over-application of patterns can make code more complex without adding value.

---

## When NOT to Use Transactions

### Skip Transactions For:

#### 1. Read-Only Operations

```typescript
// ✅ CORRECT - No transaction needed
async function getMarketDetails(id: string): Promise<Market> {
  return prisma.market.findUnique({
    where: { id },
    include: { outcomes: true, creator: true },
  });
}

// ❌ UNNECESSARY - Transaction adds no value
async function getMarketDetails(id: string): Promise<Market> {
  return prisma.$transaction(async (tx) => {
    return tx.market.findUnique({
      where: { id },
      include: { outcomes: true, creator: true },
    });
  });
}
```

#### 2. Single Atomic Database Operations

```typescript
// ✅ CORRECT - Single update is already atomic
async function incrementMarketViews(id: string): Promise<void> {
  await prisma.market.update({
    where: { id },
    data: { viewCount: { increment: 1 } },
  });
}

// ✅ ALSO FINE - Transaction doesn't hurt but adds verbosity
async function incrementMarketViews(id: string): Promise<void> {
  await prisma.$transaction(async (tx) => {
    await tx.market.update({
      where: { id },
      data: { viewCount: { increment: 1 } },
    });
  });
}
```

#### 3. Logging and Analytics (Eventual Consistency Acceptable)

```typescript
// ✅ CORRECT - Failure shouldn't block main operation
async function logMarketView(marketId: string, userId: string): Promise<void> {
  try {
    await prisma.analyticsEvent.create({
      data: { type: 'MARKET_VIEW', marketId, userId, timestamp: new Date() },
    });
  } catch (error) {
    // Log but don't fail - analytics failure shouldn't break user experience
    logger.error('Failed to log market view', error);
  }
}
```

#### 4. Idempotent Operations That Can Safely Retry

```typescript
// ✅ CORRECT - Idempotent upsert, safe to retry
async function updateUserPreferences(userId: string, prefs: Preferences): Promise<void> {
  await prisma.userPreference.upsert({
    where: { userId },
    update: { preferences: prefs },
    create: { userId, preferences: prefs },
  });
}
```

### Key Insight:

**Transactions are for ensuring consistency across multiple related changes, not for every database operation.**

---

## When NOT to Apply DRY

### Don't Extract When:

#### 1. Code Represents Different Business Concepts (Despite Similarity)

```typescript
// ✅ CORRECT - Keep separate even though similar
function validateMarketCreation(data: MarketData): void {
  if (!data.title || data.title.length < 5) {
    throw new ValidationError('Market title must be at least 5 characters');
  }
  if (data.title.length > 200) {
    throw new ValidationError('Market title cannot exceed 200 characters');
  }
}

function validateOutcomeName(name: string): void {
  if (!name || name.length < 2) {
    throw new ValidationError('Outcome name must be at least 2 characters');
  }
  if (name.length > 100) {
    throw new ValidationError('Outcome name cannot exceed 100 characters');
  }
}

// ❌ WRONG - Over-abstraction obscures different business rules
function validateString(value: string, config: ValidationConfig): void {
  if (!value || value.length < config.minLength) {
    throw new ValidationError(
      `${config.fieldName} must be at least ${config.minLength} characters`
    );
  }
  // Generic validation loses context of what's being validated
}
```

**Why:** Market titles and outcome names might evolve differently. Today they have similar validation, but tomorrow market titles might need profanity filtering while outcomes need different rules.

#### 2. Only 2 Occurrences and Unlikely to Grow

```typescript
// ✅ ACCEPTABLE - Only 2 uses, specific contexts
function calculateBinaryMarketPrice(shares: [number, number]): [number, number] {
  const total = shares[0] + shares[1];
  return [shares[0] / total, shares[1] / total];
}

// In file A:
const prices = calculateBinaryMarketPrice([yesShares, noShares]);

// In file B:
const prices = calculateBinaryMarketPrice([yesShares, noShares]);

// ❌ PREMATURE - Don't create abstraction for 2 uses
function calculateNormalizedDistribution(values: number[]): number[] {
  // Complex generic function for what might stay 2 simple uses
}
```

**Why:** Rule of Three - wait for third occurrence before extracting.

#### 3. Extraction Would Reduce Readability Significantly

```typescript
// ✅ CORRECT - Inline is clearer
async function resolveMarket(id: string, outcome: number): Promise<void> {
  await prisma.$transaction(async (tx) => {
    const market = await tx.market.findUnique({ where: { id } });
    if (!market) throw new MarketNotFoundError(id);
    if (market.resolved) throw new MarketAlreadyResolvedError(id);

    await tx.market.update({
      where: { id },
      data: { resolved: true, winningOutcome: outcome },
    });
  });
}

// ❌ WRONG - Over-abstraction makes flow harder to follow
async function resolveMarket(id: string, outcome: number): Promise<void> {
  await performMarketOperation(
    id,
    async (market, tx) => {
      await updateMarketResolution(tx, id, outcome);
    },
    { requireUnresolved: true }
  );
}
// Now you have to jump to multiple functions to understand what happens
```

**Why:** Sometimes inline code is easier to understand than abstracted code, especially for sequential operations.

#### 4. Would Require Complex Parameters or Configuration

```typescript
// ✅ CORRECT - Specific functions are clearer
function calculateBinaryMarketCost(shares: [number, number], b: number): number {
  return b * Math.log(Math.exp(shares[0] / b) + Math.exp(shares[1] / b));
}

function calculateMultiOutcomeCost(shares: number[], b: number): number {
  return b * Math.log(shares.reduce((sum, s) => sum + Math.exp(s / b), 0));
}

// ❌ WRONG - Complex configuration obscures simple operations
function calculateMarketCost(
  shares: number[],
  b: number,
  options: {
    type: 'binary' | 'multi';
    useLogarithmic: boolean;
    normalizeOutput: boolean;
    // ... more config
  }
): number {
  // Complex branching logic that's harder to understand and test
}
```

**Why:** Simple specific functions are better than complex configurable ones.

---

## When NOT to Use Dependency Injection

### Skip DI For:

#### 1. Pure Calculation Functions (No Side Effects)

```typescript
// ✅ CORRECT - No DI needed
function calculatePercentage(value: number, total: number): number {
  if (total === 0) throw new Error('Cannot divide by zero');
  return (value / total) * 100;
}

function calculateCompoundInterest(principal: number, rate: number, years: number): number {
  return principal * Math.pow(1 + rate, years);
}

// ❌ UNNECESSARY - DI adds no value
class PercentageCalculator {
  calculatePercentage(value: number, total: number): number {
    if (total === 0) throw new Error('Cannot divide by zero');
    return (value / total) * 100;
  }
}
```

**Why:** Pure functions are already testable and flexible. DI would add complexity without benefit.

#### 2. Simple Utility Functions Used in One Place

```typescript
// ✅ CORRECT - Simple utility, no abstraction needed
function formatMarketTitle(title: string): string {
  return title.trim().replace(/\s+/g, ' ');
}

class MarketService {
  async createMarket(data: MarketData): Promise<Market> {
    const formatted = formatMarketTitle(data.title);
    // ...
  }
}

// ❌ OVER-ENGINEERED
interface TitleFormatter {
  format(title: string): string;
}

class MarketTitleFormatter implements TitleFormatter {
  format(title: string): string {
    return title.trim().replace(/\s+/g, ' ');
  }
}

class MarketService {
  constructor(private titleFormatter: TitleFormatter) {}
  // Unnecessary abstraction for simple operation
}
```

**Why:** DI makes sense for dependencies you might swap or mock, not for trivial utilities.

#### 3. Configuration Objects and Constants

```typescript
// ✅ CORRECT - Direct import is fine
import { DATABASE_CONFIG } from './config';

class DatabaseService {
  private pool = createPool(DATABASE_CONFIG);
}

// ❌ UNNECESSARY - Over-injection
class DatabaseService {
  constructor(private config: DatabaseConfig) {
    this.pool = createPool(config);
  }
}
// Unless you actually need different configs per instance
```

**Why:** If configuration is static and doesn't change between instances, direct import is simpler.

#### 4. Mathematical or Domain Algorithms with Fixed Implementation

```typescript
// ✅ CORRECT - Algorithm is the implementation
function calculateLMSRPrice(shares: number[], outcome: number, b: number): number {
  const expShares = shares.map((s) => Math.exp(s / b));
  const sumExp = expShares.reduce((sum, e) => sum + e, 0);
  return expShares[outcome] / sumExp;
}

// ❌ OVER-ABSTRACTED - Unnecessary interface
interface PriceCalculator {
  calculate(shares: number[], outcome: number, b: number): number;
}

class LMSRCalculator implements PriceCalculator {
  calculate(shares: number[], outcome: number, b: number): number {
    // Same implementation as above
  }
}
```

**Why:** If you'll never swap the algorithm, the abstraction adds no value. YAGNI applies.

---

## When NOT to Follow SOLID Principles

### SRP (Single Responsibility) - When NOT to Split:

#### 1. Tightly Coupled Operations That Always Happen Together

```typescript
// ✅ CORRECT - Keep together when always used together
class MarketResolver {
  async resolve(marketId: string, outcome: number): Promise<void> {
    await prisma.$transaction(async (tx) => {
      // Validate
      const market = await tx.market.findUnique({ where: { id: marketId } });
      if (!market) throw new MarketNotFoundError(marketId);
      if (market.resolved) throw new MarketAlreadyResolvedError(marketId);

      // Update market
      await tx.market.update({
        where: { id: marketId },
        data: { resolved: true, winningOutcome: outcome },
      });

      // Calculate payouts
      const positions = await tx.position.findMany({ where: { marketId } });
      const payouts = positions.map((p) => ({
        userId: p.userId,
        amount: p.outcome === outcome ? p.shares : 0,
      }));

      // Update balances
      await Promise.all(
        payouts.map((p) =>
          tx.user.update({
            where: { id: p.userId },
            data: { balance: { increment: p.amount } },
          })
        )
      );
    });
  }
}

// ❌ OVER-SPLIT - These operations are inherently coupled
class MarketValidator {
  validateResolution() {}
}
class MarketUpdater {
  updateMarket() {}
}
class PayoutCalculator {
  calculatePayouts() {}
}
class BalanceUpdater {
  updateBalances() {}
}
class MarketResolver {
  constructor(
    private validator: MarketValidator,
    private updater: MarketUpdater,
    private calculator: PayoutCalculator,
    private balanceUpdater: BalanceUpdater
  ) {}
  // Now you have to coordinate 4 classes for one logical operation
}
```

**Why:** If operations must happen together in a transaction and share the same transaction context, splitting creates artificial complexity.

### OCP (Open/Closed) - When NOT to Abstract:

#### 1. Requirements Are Stable and Unlikely to Change

```typescript
// ✅ CORRECT - Simple direct implementation
function calculateTradeFee(amount: number): number {
  return amount * 0.01; // 1% fee
}

// ❌ PREMATURE - Abstracting stable requirement
interface FeeStrategy {
  calculate(amount: number): number;
}

class PercentageFeeStrategy implements FeeStrategy {
  constructor(private percentage: number) {}
  calculate(amount: number): number {
    return amount * this.percentage;
  }
}
// Only add this if you actually need different fee strategies
```

**Why:** Don't abstract until you actually need the flexibility. YAGNI.

### DIP (Dependency Inversion) - When NOT to Use Interfaces:

#### 1. Only One Implementation Will Ever Exist

```typescript
// ✅ CORRECT - No interface needed if only one implementation
class EmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Send email via SMTP
  }
}

class UserService {
  constructor(private emailService: EmailService) {}
}

// ❌ UNNECESSARY - Interface with single implementation
interface IEmailService {
  sendEmail(to: string, subject: string, body: string): Promise<void>;
}

class EmailService implements IEmailService {
  // Same implementation
}
// Only create interface if you might have MockEmailService, SESEmailService, etc.
```

**Why:** Interfaces are for abstraction when you need multiple implementations. If you only have one and unlikely to add more, direct dependency is simpler.

### LSP (Liskov Substitution) - When to Apply and Violations to Avoid:

**Principle:** Subtypes must be substitutable for their base types without altering program correctness.

#### Common LSP Violations to Avoid:

```typescript
// ❌ VIOLATION - Square changes Rectangle behavior
class Rectangle {
  constructor(protected width: number, protected height: number) {}

  setWidth(width: number): void {
    this.width = width;
  }

  setHeight(height: number): void {
    this.height = height;
  }

  getArea(): number {
    return this.width * this.height;
  }
}

class Square extends Rectangle {
  setWidth(width: number): void {
    this.width = width;
    this.height = width; // Breaks LSP - unexpected side effect
  }

  setHeight(height: number): void {
    this.width = height;
    this.height = height; // Breaks LSP - unexpected side effect
  }
}

// ✅ CORRECT - Use composition or separate types
interface Shape {
  getArea(): number;
}

class Rectangle implements Shape {
  constructor(private width: number, private height: number) {}
  getArea(): number { return this.width * this.height; }
}

class Square implements Shape {
  constructor(private side: number) {}
  getArea(): number { return this.side * this.side; }
}
```

#### When LSP Matters Most:

- **Collections of base types** - All items must behave consistently
- **Plugin architectures** - Plugins must honor the contract
- **Strategy patterns** - Strategies must be interchangeable

**Decision Rule:** If a subclass needs to override behavior in a way that surprises callers, don't use inheritance. Use composition or separate types.

### ISP (Interface Segregation) - When NOT to Split Interfaces:

**Principle:** Clients should not be forced to depend on methods they don't use.

#### When to Apply ISP:

```typescript
// ❌ FAT INTERFACE - Forces implementers to implement unused methods
interface Worker {
  work(): void;
  eat(): void;
  sleep(): void;
  attendMeeting(): void;
  writeReport(): void;
}

class Robot implements Worker {
  work(): void { /* OK */ }
  eat(): void { throw new Error('Robots cannot eat'); } // Forced to implement
  sleep(): void { throw new Error('Robots cannot sleep'); } // Forced to implement
  // ...
}

// ✅ SEGREGATED - Clients depend only on what they need
interface Workable {
  work(): void;
}

interface Feedable {
  eat(): void;
}

interface Restable {
  sleep(): void;
}

class Robot implements Workable {
  work(): void { /* OK */ }
}

class Human implements Workable, Feedable, Restable {
  work(): void { /* OK */ }
  eat(): void { /* OK */ }
  sleep(): void { /* OK */ }
}
```

#### When NOT to Split Interfaces:

```typescript
// ✅ CORRECT - Keep together when always used together
interface Repository<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Partial<T>): Promise<T>;
  update(id: string, data: Partial<T>): Promise<T>;
  delete(id: string): Promise<void>;
}

// ❌ OVER-SEGREGATED - Creates unnecessary complexity
interface Findable<T> {
  findById(id: string): Promise<T | null>;
  findAll(): Promise<T[]>;
}
interface Creatable<T> {
  create(data: Partial<T>): Promise<T>;
}
interface Updatable<T> {
  update(id: string, data: Partial<T>): Promise<T>;
}
interface Deletable {
  delete(id: string): Promise<void>;
}
// Over-segregation when all repos need all methods
```

**Decision Rule:** Split interfaces when different clients need different subsets. Keep together when all clients use all methods.

---

## When NOT to Apply Law of Demeter

### Law of Demeter Exceptions:

The Law of Demeter ("Don't talk to strangers") prevents tight coupling through method chaining. However, there are legitimate exceptions:

#### 1. Fluent APIs and Builder Patterns

```typescript
// ✅ CORRECT - Fluent API designed for chaining
const query = prisma.user
  .findMany({ where: { active: true } })
  .then(users => users.map(u => u.name));

const result = new QueryBuilder()
  .select('name', 'email')
  .from('users')
  .where('active', true)
  .orderBy('name')
  .build();

// ✅ CORRECT - Builder pattern
const config = new ConfigBuilder()
  .setHost('localhost')
  .setPort(3000)
  .setDebug(true)
  .build();
```

**Why:** These APIs are intentionally designed for chaining. The return type is the same object (or a wrapper), not reaching into internal structure.

#### 2. Data Transfer Objects (DTOs) and Plain Data Structures

```typescript
// ✅ ACCEPTABLE - DTO with no behavior, just data
interface ApiResponse {
  data: {
    user: {
      profile: {
        displayName: string;
      };
    };
  };
}

const displayName = response.data.user.profile.displayName;

// ✅ ACCEPTABLE - Configuration objects
const timeout = config.server.http.timeout;
```

**Why:** DTOs are passive data holders with no behavior to encapsulate. They're meant to be accessed directly.

#### 3. Standard Library and Framework Patterns

```typescript
// ✅ ACCEPTABLE - Standard array/stream operations
const names = users
  .filter(u => u.active)
  .map(u => u.name)
  .sort();

// ✅ ACCEPTABLE - Promise chains
const result = await fetchUser(id)
  .then(user => user.permissions)
  .catch(handleError);

// ✅ ACCEPTABLE - Optional chaining for null safety
const city = user?.address?.city ?? 'Unknown';
```

**Why:** These are idiomatic patterns in the language/framework. Fighting them creates worse code.

#### 4. Testing and Debugging

```typescript
// ✅ ACCEPTABLE - Test assertions often need deep access
expect(response.body.data.users[0].email).toBe('test@example.com');

// ✅ ACCEPTABLE - Debug logging
console.log(order.customer.address.formattedString);
```

**Why:** Tests need to verify internal state. Debugging needs visibility.

### When Law of Demeter DOES Apply:

```typescript
// ❌ VIOLATION - Reaching through domain objects
function calculateShipping(order: Order): number {
  const country = order.getCustomer().getAddress().getCountry().getCode();
  return shippingRates[country];
}

// ✅ CORRECT - Ask the order directly
function calculateShipping(order: Order): number {
  const country = order.getShippingCountryCode();
  return shippingRates[country];
}

// ❌ VIOLATION - Tight coupling to internal structure
function notifyUser(order: Order): void {
  const email = order.getCustomer().getContactInfo().getEmail();
  sendEmail(email, 'Order confirmed');
}

// ✅ CORRECT - Delegate notification responsibility
function notifyUser(order: Order): void {
  order.notifyCustomer('Order confirmed');
}
```

**Decision Rule:** Apply Law of Demeter when reaching through domain/business objects. Skip it for fluent APIs, DTOs, standard library patterns, and testing.

---

## When NOT to Write Tests

### Skip Tests For:

#### 1. Trivial Getters/Setters with No Logic

```typescript
// ✅ No test needed
class User {
  constructor(
    public id: string,
    public name: string
  ) {}

  getName(): string {
    return this.name;
  }
}
```

#### 2. Code That's Already Covered by Integration Tests

```typescript
// ✅ Integration test covers this
test('should create market and outcomes together', async () => {
  const market = await service.createMarket({
    title: 'Test Market',
    outcomes: ['Yes', 'No'],
  });

  expect(market.outcomes).toHaveLength(2);
});

// ❌ UNNECESSARY - Don't also unit test the outcome creation separately
// when integration test already validates the full flow
```

#### 3. Third-Party Library Wrappers (Test Your Usage, Not the Library)

```typescript
// ✅ Test your usage of the library
test('should format date correctly', () => {
  const formatted = formatDate(new Date('2024-01-01'));
  expect(formatted).toBe('January 1, 2024');
});

// ❌ UNNECESSARY - Don't test the library itself
test('should verify date-fns format function works', () => {
  // Testing the library, not your code
});
```

**Why:** Test your code, not third-party libraries. Trust that date-fns works.

---

## When NOT to Optimize

### Don't Optimize:

#### 1. Code That Runs Infrequently and Quickly

```typescript
// ✅ FINE - Runs once at startup, takes 10ms
function loadConfiguration(): Config {
  const files = readdirSync('./config');
  return files
    .filter((f) => f.endsWith('.json'))
    .map((f) => JSON.parse(readFileSync(`./config/${f}`, 'utf-8')))
    .reduce((config, file) => ({ ...config, ...file }), {});
}

// ❌ PREMATURE - Don't optimize startup code that's already fast
```

#### 2. Code That Would Become Significantly Less Readable

```typescript
// ✅ CLEAR - Easy to understand
function calculateMarketMetrics(market: Market): Metrics {
  const totalVolume = market.trades.reduce((sum, t) => sum + t.amount, 0);
  const uniqueTraders = new Set(market.trades.map((t) => t.userId)).size;
  const averageTradeSize = totalVolume / market.trades.length;

  return { totalVolume, uniqueTraders, averageTradeSize };
}

// ❌ PREMATURE - Micro-optimization that hurts readability
function calculateMarketMetrics(market: Market): Metrics {
  let totalVolume = 0;
  const traders = new Set();
  for (let i = 0, len = market.trades.length; i < len; i++) {
    const trade = market.trades[i];
    totalVolume += trade.amount;
    traders.add(trade.userId);
  }
  return {
    totalVolume,
    uniqueTraders: traders.size,
    averageTradeSize: totalVolume / market.trades.length,
  };
}
// Only do this if profiling shows the original is actually slow
```

#### 3. Without Measuring the Performance Impact

```typescript
// ❌ PREMATURE - Optimizing without data
// "I think this might be slow, let me rewrite with a more complex algorithm"

// ✅ CORRECT - Measure first
const start = performance.now();
const result = processData(largeDataset);
const duration = performance.now() - start;
console.log(`Processing took ${duration}ms`);
// Only optimize if this shows it's actually slow
```

---

## When NOT to Comment

### Don't Comment:

#### 1. What the Code Does (Make Code Self-Explanatory Instead)

```typescript
// ❌ BAD - Comment states the obvious
// Calculate the total price
const totalPrice = quantity * unitPrice;

// Loop through all markets
for (const market of markets) {
  // Process the market
  processMarket(market);
}

// ✅ GOOD - Code is self-explanatory
const totalPrice = quantity * unitPrice;

for (const market of markets) {
  processMarket(market);
}
```

#### 2. To Explain Bad Code (Fix the Code Instead)

```typescript
// ❌ BAD - Comment explains confusing code
// This gets the user's balance by finding the user and then getting their balance field
const b = users.find((u) => u.id === uid)?.b || 0;

// ✅ GOOD - Clear code needs no comment
function getUserBalance(userId: string): number {
  const user = users.find((u) => u.id === userId);
  return user?.balance ?? 0;
}
```

#### 3. Commented-Out Code (Delete It, Use Git History)

```typescript
// ❌ BAD - Commented out code
function calculatePrice(shares: number[]): number {
  // const oldImplementation = shares[0] / shares.reduce((s, sh) => s + sh, 0);
  // return oldImplementation;
  return newImplementation(shares);
}

// ✅ GOOD - Delete commented code
function calculatePrice(shares: number[]): number {
  return newImplementation(shares);
}
// If you need old implementation, check git history
```

---

## Summary: The "Less Is More" Principle

**Key Takeaways:**

1. **Transactions:** Use for multi-record changes, not every database call
2. **DRY:** Extract commonality representing same concept, not similar-looking code
3. **Dependency Injection:** Use for dependencies you'll swap/mock, not utilities
4. **SOLID:** Apply when adding flexibility, not for stable simple code
5. **Tests:** Test your logic, not trivial operations or third-party libraries
6. **Optimization:** Profile first, optimize only measured bottlenecks
7. **Comments:** Explain why and non-obvious decisions, not what
8. **Law of Demeter:** Apply to domain objects, skip for fluent APIs, DTOs, and standard patterns

**Golden Rule:** Every pattern, abstraction, and optimization should add value. If it makes code more complex without clear benefit, don't apply it.

**Remember:** The best code is code that's easy to understand, modify, and maintain. Sometimes that means fewer abstractions, not more.
