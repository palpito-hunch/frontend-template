# Development Standards Quick Reference Card

**For AI-Driven Development with Kiro IDE**

---

## 🎯 Priority Hierarchy (When Rules Conflict)

```
P0 - CRITICAL (Always First)
├── Financial Safety → Transactions, race conditions, balance consistency
└── Type Safety → Explicit types, null handling, input validation

P1 - HIGH (Unless Conflicts with P0)
├── SOLID Principles → Clean architecture, maintainability
└── DRY → Single source of truth, no duplication

P2 - MEDIUM (Only When Measured Need)
└── Performance → Profile first, optimize only bottlenecks

CROSS-CUTTING (Always Apply)
├── KISS → Keep It Simple, Stupid - simplest solution that works
├── YAGNI → You Aren't Gonna Need It - don't build for hypotheticals
├── Boy Scout Rule → Leave code cleaner than you found it
└── Law of Demeter → Don't talk to strangers - avoid method chaining
```

**Decision Rule**: When uncertain, choose safety and clarity over brevity and performance.

---

## 🔍 Before Implementation (Validate Approach)

**Before coding, briefly verify the requested approach is appropriate:**

### Quick Validation Checklist

```
1. Is this approach current?    → Check if deprecated/superseded
2. Is there a native solution?  → Platform/framework built-in option?
3. Is it the right abstraction? → Too low-level? Too complex?
4. Does it already exist?       → First-party integration available?
```

### Examples

| Request | Stop and Ask | Better Approach |
|---------|--------------|-----------------|
| "Add Slack webhook for notifications" | Is webhook the best option? | Slack GitHub App (native integration) |
| "Write a custom date parser" | Does the platform handle this? | Use `Intl.DateTimeFormat` or `date-fns` |
| "Create polling for real-time updates" | Is there a better pattern? | WebSockets, SSE, or push notifications |
| "Build custom auth system" | Is there a standard solution? | OAuth, Auth0, or framework auth |
| "Write a caching layer" | Does the framework provide this? | Redis, framework cache, CDN |

### When to Pause

- **Integrations**: Check if first-party app/integration exists
- **Infrastructure**: Check if managed service is available
- **Common patterns**: Check if framework/library handles it
- **Security-sensitive**: Check industry-standard solutions first

### How to Validate

```
1. Quick search: "[service] + [platform] integration" (e.g., "Slack GitHub integration")
2. Check official docs for recommended approach
3. Ask: "Is there a more modern/maintained way to do this?"
```

**Decision Rule**: Spend 2 minutes validating the approach to avoid hours of implementing the wrong solution.

---

## ⚡ Critical Rules (NEVER Violate)

### Financial Safety

```typescript
// ✅ ALWAYS: All balance changes in transactions
await prisma.$transaction(async (tx) => {
  const user = await tx.user.findUnique({ where: { id } }); // Fresh data
  await tx.user.update({ ... });                             // Atomic update
  await tx.trade.create({ ... });                            // Related data
});

// ❌ NEVER: Operations outside transaction
const user = await getUser(id);  // TOCTOU vulnerability!
await updateBalance(id, amount);
```

### Race Conditions

```typescript
// ✅ ALWAYS: Validation inside transaction
await prisma.$transaction(async (tx) => {
  const market = await tx.market.findUnique({ where: { id } });
  if (market.resolved) throw new Error(); // Check with fresh data
  await tx.market.update({ ... });
});

// ❌ NEVER: Check outside, use inside
const market = await getMarket(id);
if (market.resolved) throw new Error(); // Stale data!
await prisma.$transaction(async (tx) => {
  await tx.market.update({ ... }); // Race condition possible
});
```

### Variable Initialization

```typescript
// ✅ ALWAYS: Initial value for reduce()
const sum = array.reduce((acc, val) => acc + val, 0);
const max = array.reduce((m, v) => (v > m ? v : m), array[0]!);

// ❌ NEVER: reduce() without initial value
const sum = array.reduce((acc, val) => acc + val); // Breaks on empty array
```

---

## 🚨 Red Flags (Auto-Reject in Code Review)

| Pattern                                                      | Issue                          | Fix                                   |
| ------------------------------------------------------------ | ------------------------------ | ------------------------------------- |
| `await prisma.*.update()` outside `$transaction()`           | Financial operation not atomic | Wrap in transaction                   |
| `array.reduce(...)` with 1 parameter                         | Crashes on empty array         | Add initial value                     |
| `throw new Error('Invalid')`                                 | Non-actionable error           | Use specific error class with context |
| `function foo(...)` without return type                      | Type safety                    | Add `: ReturnType`                    |
| Database query in `for` loop                                 | N+1 problem                    | Use batch query with `in`             |
| `const market = await getMarket(id)` then use in transaction | TOCTOU                         | Fetch inside transaction              |

---

## ✅ ALWAYS Do

### Transactions

- ✅ All database writes in transactions
- ✅ Fetch fresh data inside transaction
- ✅ Related updates in same transaction

### Error Handling

```typescript
// ✅ Structure
throw new BusinessLogicError(
  'User balance ($10.50) insufficient for trade ($15.00)',
  'INSUFFICIENT_BALANCE',
  { userId, currentBalance, requiredAmount, shortfall }
);
```

### Types

- ✅ Explicit return types: `function foo(): ReturnType`
- ✅ Validate inputs at boundaries
- ✅ Handle null/undefined explicitly

### Testing

- ✅ Race condition tests for concurrent operations
- ✅ Property-based tests for financial calculations
- ✅ Integration tests for complex workflows

---

## ❌ When NOT to Apply

### DRY

**Skip if:**

- Different business concepts (despite similar code)
- Only 2 occurrences, unlikely to grow
- Extraction reduces clarity significantly

```typescript
// ❌ DON'T extract - different concepts
validateMarketTitle(title); // Min 5 chars
validateOutcomeName(name); // Min 2 chars
// May diverge in future
```

### Dependency Injection

**Skip if:**

- Pure calculation (no side effects)
- Simple utility, one use case
- Would add complexity without benefit

```typescript
// ❌ DON'T inject - pure function
function calculatePercentage(value: number, total: number): number {
  return (value / total) * 100;
}
```

### Transactions

**Skip if:**

- Read-only operations
- Single atomic database operation
- Logging/analytics (eventual consistency OK)

```typescript
// ✅ OK without transaction - just reading
const market = await prisma.market.findUnique({ where: { id } });
```

### Optimization

**Skip if:**

- Operation <100ms and infrequent
- Would significantly reduce clarity
- Haven't profiled to confirm bottleneck

---

## 🧹 KISS & Boy Scout Rule

### KISS (Keep It Simple, Stupid)

```typescript
// ✅ KISS - Simple and obvious
function isEligibleForDiscount(user: User): boolean {
  if (user.membershipYears >= 2) {
    return true;
  }
  return false;
}

// ❌ Over-clever
const isEligible = (u: User) => u.membershipYears >= 2 ? true : false;
// Or worse: !!(u.membershipYears >= 2)
```

### Boy Scout Rule

```typescript
// Before: You're here to fix a bug
function calc(d: any) {
  return d.x * d.y; // bug was here
}

// After: Fixed bug AND improved code
function calculateArea(dimensions: Dimensions): number {
  return dimensions.width * dimensions.height;
}
```

**Small cleanups while you work:**
- Fix typos in comments
- Rename unclear variables
- Add missing type annotations
- Remove dead code

**Don't:**
- Refactor entire modules
- Change unrelated code
- Create scope creep

### Law of Demeter (Don't Talk to Strangers)

```typescript
// ❌ VIOLATION - Chain of method calls (train wreck)
const city = order.getCustomer().getAddress().getCity().getName();

// ✅ CORRECT - Ask, don't reach through objects
const city = order.getShippingCity();

// ❌ VIOLATION - Reaching into nested structure
const card = user.getWallet().getCreditCard().getDetails();

// ✅ CORRECT - Delegate to the object
const payment = user.getPreferredPaymentMethod();
```

**Exceptions (OK to chain):**
- Fluent APIs: `query.where(...).orderBy(...).limit(10)`
- Builder patterns: `new Builder().setX(...).setY(...).build()`
- Data transfer objects (DTOs) with no behavior

---

## 🔍 Common Scenarios

### Scenario: Should I extract this duplicate code?

```
1. Same concept? NO → Don't extract
2. Used 3+ times? NO → Wait for third use
3. Reduces clarity? YES → Don't extract
4. All above pass? → Extract
```

### Scenario: Should I use a transaction?

```
1. Modifying data? NO → No transaction
2. Financial operation? YES → USE TRANSACTION
3. Multiple related changes? YES → USE TRANSACTION
4. Single atomic write? → Optional but safe
```

### Scenario: Should I optimize?

```
1. N+1 or query in loop? YES → FIX IMMEDIATELY
2. Missing index? YES → FIX IMMEDIATELY
3. Operation >100ms + frequent? → Profile first
4. Operation <100ms? → Don't optimize
```

---

## 📁 File Organization

```
src/
├── services/           # Business logic
│   ├── __tests__/
│   └── *.service.ts
├── repositories/       # Data access
│   └── *.repository.ts
├── utils/             # Pure functions
│   └── *.util.ts
├── types/             # TypeScript types
│   └── *.types.ts
└── errors/            # Custom errors
    └── *.error.ts
```

**Naming:**

- Files: `kebab-case.type.ts` (e.g., `market.service.ts`)
- Classes: `PascalCase` (e.g., `MarketService`)
- Functions: `camelCase` (e.g., `calculatePrice`)
- Constants: `SCREAMING_SNAKE_CASE` (e.g., `MAX_TRADE_AMOUNT`)
- Booleans: `isActive`, `hasBalance`, `canTrade`

**Size Limits:**

- Soft: 300 lines → Consider splitting
- Hard: 500 lines → Must split

---

## 💬 Comments

### ✅ ALWAYS Comment

- Complex algorithms (explain approach)
- Non-obvious business rules
- Race condition prevention strategies
- Performance optimizations (with measurements)
- Security considerations

```typescript
// ✅ GOOD - Explains WHY
// Fetch inside transaction to prevent TOCTOU race condition.
// Market could be resolved between check and update if fetched outside.
await prisma.$transaction(async (tx) => {
  const market = await tx.market.findUnique({ where: { id } });
  if (market.resolved) throw new Error();
});
```

### ❌ NEVER Comment

- What code does (make code self-explanatory)
- Variable declarations (use descriptive names)
- Obvious operations
- Instead of fixing bad code

```typescript
// ❌ BAD - States obvious
// Calculate total price
const totalPrice = quantity * unitPrice;
```

---

## ⚡ Performance

### ALWAYS Fix

- N+1 query problems
- Database queries in loops
- Missing indexes on frequently queried columns
- Loading unnecessary data
- O(n²) when O(n) is simple

### Profile Before Optimizing

- Operations <100ms
- Infrequently called code
- When optimization reduces clarity

### Never Optimize

- Without measuring bottleneck
- At expense of safety/correctness
- Before verifying it's actually slow

---

## 🧪 Testing Requirements

### Required Tests

- Unit tests for business logic
- Integration tests for database operations
- **Race condition tests for concurrent operations**
- Property-based tests for financial calculations

### Race Condition Test Pattern

```typescript
test('should prevent double resolution', async () => {
  const attempts = Array.from({ length: 5 }, () => service.resolveMarket(marketId, outcome));

  const results = await Promise.allSettled(attempts);
  const successes = results.filter((r) => r.status === 'fulfilled');

  expect(successes.length).toBe(1); // Only one should succeed
});
```

---

## 🔀 Git Workflow (MANDATORY)

### Branch Strategy (Gitflow with Stages)

```
main (prod)            # Production - deployed to production environment
├── uat                # UAT/Sandbox - deployed to staging for acceptance testing
│   └── develop        # Development - integration branch for features
│       ├── feature/*  # New features (e.g., feature/add-user-auth)
│       ├── fix/*      # Bug fixes (e.g., fix/balance-validation)
│       ├── refactor/* # Code refactoring
│       ├── docs/*     # Documentation changes
│       └── chore/*    # Maintenance tasks
└── hotfix/*           # Emergency fixes (branch from main, merge to all)
```

### Environment Stages

| Branch    | Environment | Purpose                          | Deploys To        |
| --------- | ----------- | -------------------------------- | ----------------- |
| `develop` | Development | Feature integration, dev testing | Dev server        |
| `uat`     | UAT/Sandbox | User acceptance testing, QA      | Staging server    |
| `main`    | Production  | Live production code             | Production server |

### Release Flow

```
feature/* → develop → uat → main
                ↓        ↓      ↓
              (dev)   (staging) (prod)
            v1.2.0-a.1  v1.2.0-rc.1  v1.2.0
```

1. **Feature branches** → PR to `develop` (squash merge)
2. **develop** → PR to `uat` (merge commit, preserves history)
3. **uat** → PR to `main` (merge commit, after QA approval)

### Version Tags

| Branch | Tag Format | Example | Purpose |
| ------ | ---------- | ------- | ------- |
| `develop` | `x.y.z-a.N` | `1.2.0-a.1` | Alpha (dev testing) |
| `uat` | `x.y.z-rc.N` | `1.2.0-rc.1` | Release candidate (QA) |
| `main` | `x.y.z` | `1.2.0` | Production release |

### Rules (NEVER Violate)

- ❌ **No direct commits to main, uat, or develop** — ALL changes via PR
- ✅ **Feature branches from develop** — Not from main or uat
- ✅ **Squash merge for features** — Clean commit per feature into develop
- ✅ **Merge commit for promotions** — develop→uat and uat→main preserve history
- ✅ **PR requires:** Passing CI + at least 1 approval
- ✅ **Delete feature branch after merge** — Keep repo clean

### Feature Development Workflow

```bash
# 1. Create feature branch from develop
git checkout develop && git pull origin develop
git checkout -b feature/my-feature

# 2. Work, commit (can be messy - will be squashed)
git commit -m "wip: initial implementation"
git commit -m "fix: address review feedback"

# 3. Push and create PR to develop
git push -u origin feature/my-feature
# Create PR: feature/my-feature → develop
# Select "Squash and merge"

# 4. After merge, clean up
git checkout develop && git pull origin develop
git branch -d feature/my-feature
```

### Stage Promotion Workflow

```bash
# Promote develop to uat (after dev testing complete)
# Create PR: develop → uat
# Select "Create a merge commit" (NOT squash)
# Requires: All tests passing + QA approval

# Promote uat to main (after UAT sign-off)
# Create PR: uat → main
# Select "Create a merge commit" (NOT squash)
# Requires: All tests passing + Product owner approval
```

### Hotfix Workflow (Emergency Only)

```bash
# 1. Branch from main
git checkout main && git pull origin main
git checkout -b hotfix/critical-bug

# 2. Fix, test, commit
git commit -m "fix: critical bug description"

# 3. PR to main (expedited review)
# After merge to main, also merge to uat and develop
git checkout uat && git merge main
git checkout develop && git merge main
```

### Commit Message (for squash commit)

```
type: short description

- Detail 1
- Detail 2

Co-Authored-By: Name <email>
```

### Merge Strategy Summary

| Merge Type        | Strategy     | Why                             |
| ----------------- | ------------ | ------------------------------- |
| feature → develop | Squash       | Clean single commit per feature |
| develop → uat     | Merge commit | Preserve feature history for QA |
| uat → main        | Merge commit | Full audit trail to production  |
| hotfix → main     | Squash       | Single fix commit               |

---

## 🧪 Testing: Mock Policy (STRICT)

### Core Principle

> **Default to REAL implementations. Mocks are a last resort, not a convenience.**

Real tests catch real bugs. Mocks hide integration issues and give false confidence.

### ❌ NEVER Mock (Use Real Implementations)

- **Database operations** — Use test database, not mocked queries
- **Business logic/services** — Test actual behavior
- **Internal service calls** — Don't mock your own code
- **Validation logic** — Test real validation rules
- **Property-based tests** — Always real implementations

### ✅ Mocks Allowed ONLY When

- **External third-party APIs** — Can't control their behavior
- **Non-deterministic behavior** — Time, randomness (use `jest.useFakeTimers()`)
- **Operations >5 seconds** — Only if truly unavoidable
- **Hard-to-trigger failures** — Network errors, disk full, etc.

### Decision Framework

```
Is it YOUR code?              YES → USE REAL IMPLEMENTATION
Is it external/third-party?   YES → Mock is acceptable
Is it non-deterministic?      YES → Mock is acceptable
Is it prohibitively slow?     YES → Mock (but consider why it's slow)
Otherwise?                    → USE REAL IMPLEMENTATION
```

### Anti-Patterns (Auto-Reject)

```typescript
// ❌ WRONG: Mocking database to avoid setup
jest.mock('@prisma/client');
prisma.user.findUnique.mockResolvedValue(fakeUser);

// ✅ CORRECT: Use real test database
beforeEach(async () => {
  testUser = await prisma.user.create({ data: testUserData });
});

// ❌ WRONG: Mocking internal services
const mockCalculator = { calculate: jest.fn().mockReturnValue(100) };

// ✅ CORRECT: Use real service
const calculator = new RealCalculator();
```

---

## 🔗 Product Development Workflow (MANDATORY)

**Product development has six phases (execute in order):**

| Phase | Name | Owner | Description |
|-------|------|-------|-------------|
| 0 | Product Brief Creation | PM | Create well-defined brief with AI assistance |
| 1 | Product Brief → Projects | PM + AI | Break down brief into Linear projects |
| 2 | Spec Creation / Refinement | PM + Engineering | Create specs with AI (super-charged Scrum refinement) |
| 3 | Spec-to-Project | AI | Populate project with issues from approved specs |
| 4 | Task Development | AI | Develop tasks with status tracking (MANDATORY) |
| 5 | Feature Verification | PM + Engineering | Verify feature before PR review |

**Key Concept**: Problem → Product Brief → Projects → Specs → Issues → Development → Verification → PR Review.

**Quality Gates:**
- Phase 0 → 1: Product brief approved
- Phase 1 → 2: Projects created in Linear
- Phase 2 → 3: PM and Engineering sign off on specs
- Phase 3 → 4: Issues created in Linear
- Phase 4 → 5: All tasks completed
- Phase 5 → PR: PM and Engineering verify feature

**SINGLE SOURCE OF TRUTH:** All workflow rules are defined in:
- `.kiro/standards/core/linear-mcp-product-to-projects.md` - Product brief → projects
- `.kiro/standards/core/linear-mcp-spec-to-project.md` - Spec files → issues
- `.kiro/standards/core/linear-mcp-task-development.md` - Task workflow (MANDATORY)
- `.kiro/standards/core/linear-mcp-rules.md` - Overview

**Violation of the task development workflow constitutes incorrect agent behavior.**

---

## 🎯 Kiro IDE Quick Commands

```bash
# Load core specs
@specs/core/priority-framework.md
@specs/core/when-not-to-apply.md
@specs/quick-reference.md

# Create service
@kiro create service --spec=feature-spec.md

# Review code
@kiro review --spec=code-review.md --strict

# Fix race conditions
@kiro fix --spec=race-conditions.md

# Optimize (after profiling!)
@kiro optimize --spec=performance.md --profile-first
```

---

## ✅ Pre-Commit Checklist

```
Before committing, verify:
[ ] On a feature branch (NOT develop/uat/main)
[ ] ESLint: 0 warnings (npm run lint:strict)
[ ] TypeScript: no errors (tsc --noEmit)
[ ] All tests passing (npm test)
[ ] Tests use real implementations (no unnecessary mocks)
[ ] All reduce() have initial values
[ ] DB writes inside transactions
[ ] Errors follow standards (message + code + context)
[ ] Functions have return types
[ ] Race condition tests for concurrent operations

Before creating PR to develop:
[ ] Branch is up to date with develop
[ ] Commits are ready to be squashed
[ ] PR description explains the change
[ ] PR targets develop branch (not uat or main)
```

---

## 🆘 When in Doubt

### Decision Framework

1. **Safety first** → Choose safer option
2. **Explicit over implicit** → Add null checks
3. **Clarity over brevity** → Write clearer code
4. **Simple over complex** → Choose simpler approach
5. **Measure over guess** → Profile before optimizing
6. **Ask** → Clarify business requirements

### Quick Decisions

- **Unknown nullability?** → Add explicit check
- **Unsure about transaction?** → Use transaction (safer)
- **DRY or not?** → Wait for third occurrence
- **Optimize or not?** → Profile first
- **Comment or not?** → Only if explains WHY

---

## 📚 Standards Documents Map

**Core (Load Always)**

- `quick-reference.md` - One-page consolidated reference (this file)
- `priority-framework.md` - Decision rules when standards conflict
- `when-not-to-apply.md` - When to skip patterns (SOLID, DRY, etc.)
- `epistemic-honesty.md` - Cite sources, flag gaps, don't speculate
- `linear-mcp-rules.md` - Linear MCP integration overview
- `linear-mcp-product-to-projects.md` - Product brief → projects (FIRST STEP)
- `linear-mcp-spec-to-project.md` - Spec files → issues
- `linear-mcp-task-development.md` - Task development workflow (MANDATORY)

**Domain Standards**

- `git-workflow.md` - Branching, PRs, squash merge (MANDATORY)
- `testing-mocks.md` - Mock policy (real implementations first)
- `errors.md` - Error handling patterns
- `file-organization.md` - Project structure
- `comments.md` - Documentation standards
- `performance.md` - Optimization guidelines

**Workflows**

- `code-review-checklist.md` - Review process
- `race-conditions.md` - Concurrent safety
- `testing.md` - Testing guidelines

---

## 🎓 Remember

**The Mantra:**

> "Make it work, make it right, make it fast - in that order"

**The Priority:**

> Safety > Correctness > Maintainability > Performance

**The Goal:**

> Code that's easy to understand, modify, and doesn't lose money

---

**Print this card and keep it visible during development!**
