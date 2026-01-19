# Performance Optimization Standards

## Purpose

Performance optimization is important, but premature optimization wastes time and creates complex code. This guide defines when and how to optimize.

---

## Core Principle

**"Premature optimization is the root of all evil" - Donald Knuth**

**Corollary:** "But we should not pass up obvious opportunities in the name of avoiding premature optimization"

---

## When to Optimize

### ✅ ALWAYS Fix These (Obvious Inefficiencies)

#### 1. N+1 Query Problems

**❌ BAD - N+1 Queries**

```typescript
async function getMarketsWithCreators(): Promise<MarketWithCreator[]> {
  const markets = await prisma.market.findMany();

  // Creates N+1 queries: 1 for markets, then 1 per market for creator
  return Promise.all(
    markets.map(async (market) => ({
      ...market,
      creator: await prisma.user.findUnique({
        where: { id: market.creatorId },
      }),
    }))
  );
}
```

**✅ GOOD - Single Query with Include**

```typescript
async function getMarketsWithCreators(): Promise<MarketWithCreator[]> {
  // Single query with join
  return prisma.market.findMany({
    include: { creator: true },
  });
}
```

#### 2. Database Queries in Loops

**❌ BAD - Query Per Iteration**

```typescript
async function getUserBalances(userIds: string[]): Promise<number[]> {
  const balances = [];

  // One query per user - very slow for large arrays
  for (const userId of userIds) {
    const user = await prisma.user.findUnique({
      where: { id: userId },
      select: { balance: true },
    });
    balances.push(user?.balance ?? 0);
  }

  return balances;
}
```

**✅ GOOD - Batch Query**

```typescript
async function getUserBalances(userIds: string[]): Promise<number[]> {
  // Single batch query
  const users = await prisma.user.findMany({
    where: { id: { in: userIds } },
    select: { id: true, balance: true },
  });

  // Create lookup map for O(1) access
  const balanceMap = new Map(users.map((u) => [u.id, u.balance]));

  // Return in original order
  return userIds.map((id) => balanceMap.get(id) ?? 0);
}
```

#### 3. Missing Database Indexes

**Always add indexes for:**

- Foreign keys
- Frequently filtered columns (WHERE clauses)
- Frequently sorted columns (ORDER BY)
- Frequently joined columns

```sql
-- ✅ Add indexes for common queries
CREATE INDEX idx_markets_creator_id ON markets(creator_id);
CREATE INDEX idx_trades_market_id ON trades(market_id);
CREATE INDEX idx_trades_user_id ON trades(user_id);
CREATE INDEX idx_markets_created_at ON markets(created_at);
CREATE INDEX idx_markets_status ON markets(status);

-- Composite index for common query pattern
CREATE INDEX idx_markets_status_created ON markets(status, created_at);
```

#### 4. Loading Unnecessary Data

**❌ BAD - Load Everything**

```typescript
async function getMarketTitles(): Promise<string[]> {
  // Loads all columns when we only need title
  const markets = await prisma.market.findMany();
  return markets.map((m) => m.title);
}
```

**✅ GOOD - Select Only What You Need**

```typescript
async function getMarketTitles(): Promise<string[]> {
  // Only loads title column
  const markets = await prisma.market.findMany({
    select: { title: true },
  });
  return markets.map((m) => m.title);
}
```

#### 5. Obvious Algorithmic Inefficiencies

**❌ BAD - O(n²) When O(n) Possible**

```typescript
function findDuplicates(items: string[]): string[] {
  const duplicates = [];

  // O(n²) - nested loops
  for (let i = 0; i < items.length; i++) {
    for (let j = i + 1; j < items.length; j++) {
      if (items[i] === items[j]) {
        duplicates.push(items[i]);
      }
    }
  }

  return duplicates;
}
```

**✅ GOOD - O(n) Using Set**

```typescript
function findDuplicates(items: string[]): string[] {
  const seen = new Set<string>();
  const duplicates = new Set<string>();

  // O(n) - single pass
  for (const item of items) {
    if (seen.has(item)) {
      duplicates.add(item);
    }
    seen.add(item);
  }

  return Array.from(duplicates);
}
```

---

### ⚠️ Profile Before Optimizing

#### When to Profile First

- Operations taking <100ms
- Infrequently called code (startup, admin operations)
- Code that's already "fast enough" for current load
- Optimizations that significantly reduce readability

#### How to Profile

**Node.js Built-in Profiling:**

```typescript
// Simple timing
const start = performance.now();
const result = await expensiveOperation();
const duration = performance.now() - start;

console.log(`Operation took ${duration.toFixed(2)}ms`);

// Profile with real data
if (process.env.NODE_ENV === 'development') {
  console.time('calculateMarketPrices');
  const prices = await calculateMarketPrices(markets);
  console.timeEnd('calculateMarketPrices');
}
```

**Load Testing:**

```typescript
// Use tools like autocannon, k6, or artillery
import autocannon from 'autocannon';

autocannon(
  {
    url: 'http://localhost:3000/api/markets',
    connections: 100,
    duration: 10,
    pipelining: 1,
  },
  (err, result) => {
    console.log('Requests/sec:', result.requests.mean);
    console.log('Latency p99:', result.latency.p99);
  }
);
```

---

### ❌ DON'T Optimize When

#### 1. Operation is Already Fast

```typescript
// ✅ FINE - Takes 2ms, don't optimize
function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD',
  }).format(amount);
}

// ❌ UNNECESSARY - Micro-optimization adds complexity
const currencyFormatter = new Intl.NumberFormat('en-US', {
  style: 'currency',
  currency: 'USD',
});

function formatCurrency(amount: number): string {
  return currencyFormatter.format(amount);
}
// Only worth it if profiling shows formatCurrency is a bottleneck
```

#### 2. Code is Called Infrequently

```typescript
// ✅ FINE - Runs once at startup, takes 50ms
function loadConfiguration(): Config {
  const files = readdirSync('./config');
  return files
    .filter((f) => f.endsWith('.json'))
    .map((f) => JSON.parse(readFileSync(`./config/${f}`, 'utf-8')))
    .reduce((config, file) => ({ ...config, ...file }), {});
}

// Startup code doesn't need optimization unless it's actually slow
```

#### 3. Optimization Significantly Reduces Clarity

```typescript
// ✅ CLEAR - Easy to understand
function calculateMarketStats(trades: Trade[]): MarketStats {
  const totalVolume = trades.reduce((sum, t) => sum + t.amount, 0);
  const uniqueTraders = new Set(trades.map((t) => t.userId)).size;
  const avgTradeSize = totalVolume / trades.length;

  return { totalVolume, uniqueTraders, avgTradeSize };
}

// ❌ PREMATURE - Micro-optimized but harder to read
function calculateMarketStats(trades: Trade[]): MarketStats {
  let totalVolume = 0;
  const traders = new Set<string>();

  for (let i = 0, len = trades.length; i < len; i++) {
    const trade = trades[i];
    totalVolume += trade.amount;
    traders.add(trade.userId);
  }

  return {
    totalVolume,
    uniqueTraders: traders.size,
    avgTradeSize: totalVolume / trades.length,
  };
}
// Only do this if profiling proves the first version is too slow
```

---

## Performance Testing Requirements

### Required Performance Tests

#### 1. Database Query Performance

```typescript
describe('Market Query Performance', () => {
  test('should fetch 1000 markets with creators in <500ms', async () => {
    const start = performance.now();

    const markets = await prisma.market.findMany({
      take: 1000,
      include: { creator: true },
    });

    const duration = performance.now() - start;

    expect(duration).toBeLessThan(500);
    expect(markets).toHaveLength(1000);
  });
});
```

#### 2. Concurrent Load Performance

```typescript
describe('Concurrent Trade Performance', () => {
  test('should handle 100 concurrent trades within 5s', async () => {
    const trades = Array.from({ length: 100 }, (_, i) => ({
      userId: `user-${i}`,
      marketId: testMarketId,
      amount: 10,
      outcome: 0,
    }));

    const start = performance.now();

    const results = await Promise.allSettled(trades.map((t) => tradeService.placeTrade(t)));

    const duration = performance.now() - start;

    expect(duration).toBeLessThan(5000);

    const successes = results.filter((r) => r.status === 'fulfilled');
    expect(successes.length).toBeGreaterThan(90); // >90% success rate
  });
});
```

#### 3. Large Dataset Performance

```typescript
describe('Large Dataset Performance', () => {
  test('should calculate prices for 1000 outcomes in <1s', async () => {
    const shares = Array.from({ length: 1000 }, () => Math.random() * 100);

    const start = performance.now();

    const prices = shares.map((_, i) => calculateLMSRPrice(shares, i, 250));

    const duration = performance.now() - start;

    expect(duration).toBeLessThan(1000);
    expect(prices).toHaveLength(1000);
  });
});
```

---

## Common Performance Patterns

### 1. Caching

**When to cache:**

- Data that's expensive to compute
- Data that changes infrequently
- Data that's accessed frequently

**When NOT to cache:**

- Data that changes frequently (user balances, market prices)
- Data that's cheap to compute
- Before measuring it's actually a bottleneck

**Example:**

```typescript
// ✅ GOOD - Cache expensive, static data
class MarketService {
  private marketConfigCache = new Map<string, MarketConfig>();

  async getMarketConfig(type: string): Promise<MarketConfig> {
    // Check cache first
    if (this.marketConfigCache.has(type)) {
      return this.marketConfigCache.get(type)!;
    }

    // Fetch and cache
    const config = await prisma.marketConfig.findUnique({
      where: { type },
    });

    this.marketConfigCache.set(type, config);
    return config;
  }
}
```

### 2. Pagination

**Always paginate large result sets:**

```typescript
// ✅ GOOD - Paginated
async function getMarkets(
  page: number = 1,
  pageSize: number = 50
): Promise<PaginatedResult<Market>> {
  const skip = (page - 1) * pageSize;

  const [markets, total] = await Promise.all([
    prisma.market.findMany({
      skip,
      take: pageSize,
      orderBy: { createdAt: 'desc' },
    }),
    prisma.market.count(),
  ]);

  return {
    data: markets,
    page,
    pageSize,
    total,
    totalPages: Math.ceil(total / pageSize),
  };
}
```

### 3. Lazy Loading

**Load data only when needed:**

```typescript
class Market {
  private _outcomes?: Outcome[];

  // ✅ Lazy load outcomes only when accessed
  async getOutcomes(): Promise<Outcome[]> {
    if (!this._outcomes) {
      this._outcomes = await prisma.outcome.findMany({
        where: { marketId: this.id },
      });
    }
    return this._outcomes;
  }
}
```

### 4. Batch Operations

**Process multiple items together:**

```typescript
// ❌ BAD - Update one at a time
async function updateUserBalances(updates: BalanceUpdate[]): Promise<void> {
  for (const update of updates) {
    await prisma.user.update({
      where: { id: update.userId },
      data: { balance: update.newBalance },
    });
  }
}

// ✅ GOOD - Batch update
async function updateUserBalances(updates: BalanceUpdate[]): Promise<void> {
  await prisma.$transaction(
    updates.map((update) =>
      prisma.user.update({
        where: { id: update.userId },
        data: { balance: update.newBalance },
      })
    )
  );
}
```

---

## Performance Monitoring

### Required Metrics

**Track these metrics:**

- API endpoint response times (p50, p95, p99)
- Database query durations
- Cache hit rates
- Error rates
- Concurrent user load

**Implementation:**

```typescript
// Middleware for response time tracking
app.use((req, res, next) => {
  const start = performance.now();

  res.on('finish', () => {
    const duration = performance.now() - start;

    logger.info('Request completed', {
      method: req.method,
      path: req.path,
      statusCode: res.statusCode,
      duration: duration.toFixed(2),
      timestamp: new Date().toISOString(),
    });

    // Send to monitoring service
    metrics.recordResponseTime(req.path, duration);
  });

  next();
});
```

---

## Performance Budgets

### Set Performance Targets

**API Endpoints:**

- List operations: <200ms (p95)
- Detail operations: <100ms (p95)
- Write operations: <500ms (p95)
- Complex calculations: <1s (p95)

**Database Queries:**

- Simple queries: <50ms
- Joins with indexes: <100ms
- Aggregations: <200ms

**Page Load:**

- Time to Interactive: <3s
- First Contentful Paint: <1.5s

### Alert on Violations

```typescript
// Alert if response time exceeds budget
if (duration > PERFORMANCE_BUDGET[endpoint]) {
  logger.warn('Performance budget exceeded', {
    endpoint,
    duration,
    budget: PERFORMANCE_BUDGET[endpoint],
    overage: duration - PERFORMANCE_BUDGET[endpoint],
  });

  // Send alert to monitoring
  alerting.send({
    severity: 'warning',
    message: `${endpoint} exceeded performance budget`,
  });
}
```

---

## Summary: Performance Optimization Checklist

### Before Optimizing:

- [ ] Have you profiled to identify the actual bottleneck?
- [ ] Is this operation actually slow enough to matter?
- [ ] Is it called frequently enough to justify optimization?
- [ ] Will optimization significantly reduce code clarity?

### Always Fix:

- [ ] N+1 query problems
- [ ] Database queries in loops
- [ ] Missing indexes on frequently queried columns
- [ ] Loading unnecessary data
- [ ] O(n²) algorithms where O(n) is simple

### Consider Optimizing:

- [ ] Operations >100ms called frequently
- [ ] Queries returning >1000 records
- [ ] API endpoints >500ms response time
- [ ] Memory leaks or unbounded growth

### Don't Optimize:

- [ ] Operations <100ms called infrequently
- [ ] Without profiling data
- [ ] When it significantly reduces clarity
- [ ] Before measuring impact

### Monitor:

- [ ] Response time percentiles (p50, p95, p99)
- [ ] Database query performance
- [ ] Error rates under load
- [ ] Resource utilization

**Remember:** "Make it work, make it right, make it fast" - in that order. Correctness and maintainability come before performance.
