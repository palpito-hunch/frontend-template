# TDD Workflow Standards

Test-Driven Development: Write tests first, then implement.

## Core Cycle

```
RED → GREEN → REFACTOR
 ↓       ↓        ↓
Write  Make it  Clean up
failing  pass   (tests stay green)
test
```

## Principles

1. **Tests define the contract** — write what you want before how to get it
2. **One test = one behavior** — matches task granularity rules
3. **Verify failure first** — a test that can't fail is worthless
4. **Small steps** — tight feedback loops catch errors early
5. **Isolation** — tests run independently in any order

## Prerequisites

Before writing service/endpoint tests, ensure these exist:

### Scaffold Phase (Section 0)

```markdown
- [ ] 0.1 Initialize project with language/framework
- [ ] 0.2 Configure linting and formatting
- [ ] 0.3 Configure test framework
- [ ] 0.4 Configure test coverage thresholds
- [ ] 0.5 Initialize database/ORM with test database
- [ ] 0.6 Create environment configuration
- [ ] 0.7 Configure test database setup/teardown
- [ ] 0.8 Create custom error classes
- [ ] 0.9 Create test data factories
- [ ] 0.10 Checkpoint: Scaffold complete
```

### Linting and Formatting Configuration

Linting must be configured early to catch issues before tests are written.

```markdown
- [ ] 0.2 Configure linting and formatting
  - Install linter (ESLint, Biome, etc.) and formatter (Prettier, etc.)
  - Configure rules to match project coding standards
  - Verify no conflicts between linter, formatter, and compiler
  - Add lint/format scripts to package.json
  - **Depends on:** 0.1
  - **Verify:** `lint` and `format` commands run without errors on empty project
```

#### Rule Alignment Checklist

- [ ] Linting rules reflect documented coding standards
- [ ] Formatter config doesn't conflict with linter (e.g., semicolons, quotes)
- [ ] No rules that contradict each other across tools

#### TypeScript vs ESLint: Division of Responsibility

**Prefer TypeScript over ESLint when both can enforce the same rule.** TypeScript errors are caught at compile time and integrate better with IDEs.

| Responsibility | Tool | Examples |
|----------------|------|----------|
| Type checking | TypeScript | Type errors, type inference |
| Unused variables | TypeScript | `noUnusedLocals`, `noUnusedParameters` |
| Implicit any | TypeScript | `noImplicitAny` |
| Null checks | TypeScript | `strictNullChecks` |
| Unreachable code | TypeScript | `allowUnreachableCode: false` |
| Code style | ESLint | Formatting, spacing, quotes |
| Complexity limits | ESLint | `complexity`, `max-lines-per-function` |
| Naming conventions | ESLint | `@typescript-eslint/naming-convention` |
| Import ordering | ESLint | `import/order`, `sort-imports` |
| Framework rules | ESLint | React hooks, Angular rules, etc. |

#### Disable ESLint Rules That Duplicate TypeScript

When using `@typescript-eslint`, disable base ESLint rules that have TypeScript equivalents:

```json
{
  "rules": {
    // Disable base rules
    "no-unused-vars": "off",
    "no-undef": "off",
    "no-redeclare": "off",
    "no-shadow": "off",

    // Use TypeScript-aware versions only if you need ESLint's extra config
    // Otherwise, prefer tsconfig options
  }
}
```

#### Exception: ESLint for Granular Configuration

Some teams prefer ESLint for `no-unused-vars` because it offers more granular configuration (like ignoring variables prefixed with `_`):

```json
{
  "rules": {
    "no-unused-vars": "off",
    "@typescript-eslint/no-unused-vars": ["error", {
      "argsIgnorePattern": "^_",
      "varsIgnorePattern": "^_",
      "destructuredArrayIgnorePattern": "^_"
    }]
  }
}
```

If using this exception, disable the TypeScript equivalents:

```json
// tsconfig.json
{
  "compilerOptions": {
    "noUnusedLocals": false,
    "noUnusedParameters": false
  }
}
```

**Choose one tool per rule — never both.**

#### Recommended TypeScript Configuration

```json
// tsconfig.json - TypeScript handles type-related checks
{
  "compilerOptions": {
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "allowUnreachableCode": false,
    "noFallthroughCasesInSwitch": true,
    "noImplicitReturns": true
  }
}
```

```json
// eslint.config.js or .eslintrc - ESLint handles style and complexity
{
  "rules": {
    // Disable rules TypeScript handles
    "no-unused-vars": "off",
    "no-undef": "off",

    // ESLint-only concerns
    "@typescript-eslint/explicit-function-return-type": ["error", { "allowExpressions": true }],
    "@typescript-eslint/no-floating-promises": "error",
    "@typescript-eslint/naming-convention": "error",
    "complexity": ["error", 10],
    "import/order": "error"
  }
}
```

#### Common Linter/Formatter Conflicts

| Conflict | Symptom | Fix |
|----------|---------|-----|
| Prettier + ESLint formatting | Lint errors after format | Use `eslint-config-prettier` to disable ESLint formatting rules |
| Tabs vs spaces | Constant reformatting | Align `.editorconfig`, Prettier, and ESLint |
| Trailing commas | Lint/format fight | Set same value in both Prettier and ESLint |
| Quote style | Constant changes | Set same value in both tools |
| Semicolons | Constant changes | Set same value in both tools |

#### Verification

Before proceeding past scaffold:

```bash
# All commands should pass with no errors/warnings
lint .
format --check .
tsc --noEmit  # TypeScript projects
```

If any tool reports errors that another tool's config creates, resolve the conflict before writing code.

### Environment Configuration

Test environment must:
- Use a dedicated test database (never production)
- Include safety checks to prevent running against non-test databases
- Validate all required environment variables on startup

### Test Database Setup

- Reset database state in `beforeEach` (not `beforeAll`)
- Handle foreign key constraints when truncating tables
- Disconnect cleanly in `afterAll`

### Custom Error Classes

Create domain-specific errors with:
- Descriptive message
- HTTP status code
- Error name/type

Common errors: ConflictError (409), UnauthorizedError (401), ForbiddenError (403), NotFoundError (404), ValidationError (400)

### Test Data Factories

- Use deterministic data (no `Math.random()`)
- Support overrides for customization
- Use incrementing counters for unique values

```
createEntityDto(overrides?) → valid input object
createTestEntity(overrides?) → valid domain object
```

## Task Ordering

Tests come BEFORE implementations. Always.

### Correct Order

```markdown
- [ ] 4.1 Create Service class skeleton
- [ ] 4.2 Create Service test file
- [ ] 4.3 Write test: operation succeeds with valid input
- [ ] 4.4 Implement operation
- [ ] 4.5 Write test: operation rejects invalid input
- [ ] 4.6 Handle invalid input in operation
```

### Wrong Order

```markdown
- [ ] 4.1 Create Service class skeleton
- [ ] 4.2 Implement operation
- [ ] 4.3 Write test for operation  # TOO LATE
```

## Task Pairing Pattern

Each behavior follows: **Skeleton → Test File → Test → Implementation**

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Skeleton   │ ──→ │ Test File   │ ──→ │ Write Test  │ ──→ │ Implement   │
│  (once)     │     │ (once)      │     │ (RED)       │     │ (GREEN)     │
└─────────────┘     └─────────────┘     └─────────────┘     └─────────────┘
                                               ↓                   │
                                        Verify: fails              │
                                        for expected reason        │
                                               ↓                   ↓
                                             ┌───────────────────────┐
                                             │ Verify: ALL tests pass│
                                             └───────────────────────┘
                                                        ↓
                                                   ┌─────────┐
                                                   │ Commit  │
                                                   └─────────┘
```

Repeat Test → Implementation for each behavior.

## Skeleton Requirements

Skeletons allow tests to import and call methods that will fail predictably.

### Skeleton checklist

- [ ] Class/module file exists
- [ ] Constructor accepts dependencies
- [ ] Method signatures defined with correct types
- [ ] Methods throw `Error('Not implemented')`
- [ ] File imports without syntax errors

## Test File Setup

Create test file before writing first test.

- Import service under test
- Import test setup (database, factories)
- Set up describe blocks for each public method
- Verify: runs with 0 tests, no errors

## Test Isolation

Each test must be independent:

- No shared mutable state between tests
- No reliance on execution order
- Each test sets up its own data (use factories)
- `beforeEach` resets database, not `beforeAll`

### Verify isolation

```bash
# Run single test
test -t "test name"

# Run full suite — should have same result
test path/to/file.test.ts

# Run in random order
test --shuffle
```

If tests pass individually but fail together → isolation problem.

### Common isolation issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| Test B fails after Test A | Shared DB state | Reset in beforeEach |
| Random failures | Non-deterministic data | Use seeded factories |
| Works locally, fails CI | Environment differences | Use .env.test consistently |

## RED Phase: Writing Tests

### Task structure

One test task = one behavior. Apply split signals from granularity rules.

```markdown
# ❌ Bundled (violates "> 3 sub-bullets" rule)
- [ ] 4.3 Write tests for entity creation
  - Test successful creation
  - Test duplicate rejected
  - Test invalid input rejected
  - Test missing field rejected

# ✅ Split (one behavior per task)
- [ ] 4.3 Write test: creation succeeds with valid input
- [ ] 4.7 Write test: creation rejects duplicate
- [ ] 4.9 Write test: creation rejects invalid input
- [ ] 4.11 Write test: creation rejects missing field
```

### Test task anatomy

```markdown
- [ ] 4.3 Write test: creation succeeds with valid input
  - Input: valid DTO (use factory)
  - Assert: returns entity with expected fields
  - Assert: sensitive fields excluded from response
  - Assert: entity persisted to database
  - **Depends on:** test file, DTO schema
  - **Verify:** test command — 1 failing test (NotImplemented)
```

### Test naming convention

Pattern: `[expected behavior] [condition]`

```
describe('Service', () => {
  describe('operation', () => {
    it('succeeds with valid input', ...)
    it('throws ConflictError when duplicate exists', ...)
    it('throws ValidationError when input invalid', ...)
  })
})
```

Task title should align with test description:

| Task | Test block |
|------|------------|
| Write test: creation succeeds | `it('succeeds with valid input', ...)` |
| Write test: creation rejects duplicate | `it('throws ConflictError when duplicate exists', ...)` |

### Assertion quality

#### ❌ Weak assertions

```
it('creates entity', async () => {
  const result = await service.create(dto)
  expect(result).toBeDefined()  // Too vague
  expect(result).toBeTruthy()   // Meaningless
})
```

#### ✅ Strong assertions

```
it('creates entity with valid input', async () => {
  const dto = createEntityDto({ name: 'Test' })
  const result = await service.create(dto)

  // Assert return value structure and types
  expect(result.id).toMatch(/^[0-9a-f-]{36}$/)  // UUID format
  expect(result.name).toBe('Test')              // Exact value
  expect(result.createdAt).toBeInstanceOf(Date) // Correct type

  // Assert excluded fields
  expect(result).not.toHaveProperty('sensitiveField')

  // Assert side effects (persistence)
  const dbRecord = await db.entity.findUnique({ where: { id: result.id } })
  expect(dbRecord).not.toBeNull()
})
```

#### Assertion checklist

- [ ] Assert return value structure
- [ ] Assert specific field values (not just existence)
- [ ] Assert correct types
- [ ] Assert side effects (database writes)
- [ ] Assert excluded/forbidden fields

### Error assertion standards

Test both error type AND relevant details:

```
it('throws ConflictError when duplicate exists', async () => {
  const dto = createEntityDto({ name: 'taken' })
  await service.create(dto)

  // Assert error type
  await expect(service.create(dto))
    .rejects
    .toThrow(ConflictError)

  // Assert error details
  try {
    await service.create(dto)
    expect.fail('Should have thrown')
  } catch (error) {
    expect(error).toBeInstanceOf(ConflictError)
    expect(error.message).toContain('name')
    expect(error.statusCode).toBe(409)
  }
})
```

### RED phase verification

After writing a test, agent MUST:

1. Run the test
2. Confirm it **fails for the expected reason**
3. Check failure mode:

| Outcome | Meaning | Action |
|---------|---------|--------|
| Test fails with `Error: Not implemented` | ✅ Correct | Proceed to GREEN |
| Test fails with expected assertion | ✅ Correct | Proceed to GREEN |
| Test passes | ❌ Test is wrong | Fix the test |
| Test errors (import/syntax/timeout) | ❌ Setup incomplete | Fix skeleton or test file |
| Test fails with unexpected reason | ⚠️ Wrong assumption | Review test logic |

## GREEN Phase: Implementation

### Task structure

```markdown
- [ ] 4.4 Implement entity creation
  - Validate input
  - Persist to database
  - Return response (exclude sensitive fields)
  - **Depends on:** 4.3
  - **Verify:** test command — ALL tests pass
  - **Commit:** Yes
```

### GREEN phase rules

1. **Minimal code** — only enough to pass the test
2. **No premature optimization**
3. **Don't add untested behavior**
4. **Run ALL tests after implementation** — not just the new one

### Verification runs full test file

```bash
# ✅ Correct — catches regressions
test path/to/service.test.ts

# ❌ Wrong — misses regressions
test -t "specific test name"
```

## Regression Handling

If implementation breaks a previous test:

1. **Stop** — don't proceed to next task
2. **Diagnose** — which change caused regression?
3. **Fix** — restore green state before continuing
4. **Verify** — ALL tests pass, not just current one

```markdown
# If 4.6 breaks test from 4.3:
- [ ] 4.6 Handle duplicate in creation  ← BLOCKED: regression in 4.3
  - Issue: [describe what broke]
  - Fix: [describe resolution]
  - **Verify:** ALL tests pass before marking complete
```

## Setup vs Test Failures

| Error Location | Symptom | Action |
|----------------|---------|--------|
| `beforeAll`/`beforeEach` | All tests fail/skip | Fix setup first |
| `afterEach`/`afterAll` | Tests pass but cleanup errors | Fix teardown |
| Single test block | One test fails | Fix that test |
| Import/require | Suite won't load | Fix imports/syntax |
| Environment | Connection errors | Check test env config |

## Stuck in RED (Can't Reach GREEN)

If agent cannot make test pass after 2-3 attempts:

1. **Re-read test** — is the assertion correct?
2. **Check dependencies** — is skeleton/model/DTO correct?
3. **Check test setup** — is database state correct?
4. **Simplify** — can test be made less strict temporarily?
5. **Flag for review** — add `BLOCKED:` prefix, note issue, continue with next independent task

```markdown
- [ ] 4.4 Implement entity creation  ← BLOCKED: dependency issue
  - Issue: Cannot resolve module
  - Attempted: checked imports, verified installation
  - Next: Need human review
```

## REFACTOR Phase

Optional cleanup after GREEN. Tests must stay green.

```markdown
- [ ] 4.16 Refactor: extract utility function
  - Move logic to shared utility
  - Update service to use utility
  - **Depends on:** checkpoint
  - **Verify:** all tests pass, no changes to test assertions
  - **Commit:** Yes
```

### Refactor rules

1. **Only after tests pass**
2. **Tests stay green throughout** — run after each change
3. **No new behavior** — new behavior requires new tests first
4. **Don't modify test assertions** — only implementation

## Testing Private Methods

**Don't test private methods directly.** Test through public interface.

### ❌ Wrong

```
// Accessing private method
it('hashes correctly', () => {
  const hash = service['privateMethod']('input')  // Don't do this
})
```

### ✅ Correct

```
// Test through public behavior
it('stores hashed value', async () => {
  const dto = createEntityDto({ secret: 'plaintext' })
  await service.create(dto)

  const dbRecord = await db.entity.findFirst()
  expect(dbRecord.secretHash).not.toBe('plaintext')  // Not plaintext
})
```

If a private method is complex enough to need direct testing, extract it to a utility and test as a unit.

## Handling Flaky Tests

### Symptoms

- Test passes locally, fails in CI
- Test passes sometimes, fails other times
- Test fails only when run with other tests

### Common causes and fixes

| Cause | Fix |
|-------|-----|
| Timing/race conditions | Add proper await, don't use arbitrary delays |
| Shared state | Ensure test isolation, reset in beforeEach |
| Order dependence | Each test sets up own data |
| Time-dependent logic | Mock time with test framework utilities |
| Random data | Use deterministic factories (no Math.random) |
| Async cleanup not awaited | Await all cleanup in afterEach |

### If flaky test found

1. **Do not skip** — flaky tests hide real bugs
2. Reproduce locally: run test multiple times in loop
3. Identify cause from table above
4. Fix root cause, not symptoms
5. Verify: run multiple times with no failures

## Test Speed Tiers

### Organize by speed

```
src/
  services/
    entity.service.ts
    entity.service.test.ts        # Fast: unit + light integration
  test/
    integration/
      entity.integration.test.ts  # Slow: full HTTP stack
```

### Task verification uses appropriate tier

```markdown
# During RED/GREEN cycle — fast tests only
- **Verify:** fast test command

# At checkpoints — all tests including integration
- **Verify:** full test command — all pass
```

## Test Scope

| Layer | Test Type | What's Real | What's Mocked |
|-------|-----------|-------------|---------------|
| Service | Integration | DB, other services | External APIs, time, randomness |
| Endpoint | Integration | Full HTTP stack, DB | External APIs, time, randomness |
| Utility | Unit | Nothing | Everything (pure functions) |

Default to integration tests. Unit tests only for pure utility functions.

### Mock boundaries

```
// ✅ Real database
const entity = await db.entity.create({ data: entityData })

// ✅ Real service calls
const result = await service.create(dto)

// ✅ Mock external APIs
mock('../lib/external-api', () => ({
  call: mockFn().mockResolvedValue({ id: '123' })
}))

// ✅ Mock time
useFakeTimers()
setSystemTime(new Date('2024-01-15'))

// ✅ Mock randomness
spyOn(Math, 'random').mockReturnValue(0.5)
```

## Commit Points

### When to commit

- After each GREEN (test passes)
- Before starting REFACTOR
- After REFACTOR complete

### Commit message format

```
[TDD] 4.4 Implement entity creation

- Add input validation
- Persist to database
- Return sanitized response

Tests: N passing
```

### Task notation

```markdown
- [ ] 4.4 Implement entity creation
  - Validate input
  - Persist to database
  - **Depends on:** 4.3
  - **Verify:** test command — all pass
  - **Commit:** Yes
```

## Checkpoints

Verify all tests pass before proceeding to next module.

```markdown
- [ ] 4.17 Checkpoint: Service complete
  - All service tests passing
  - No skipped tests
  - Coverage meets threshold
  - **Depends on:** 4.1–4.16
  - **Verify:** coverage command — all pass, thresholds met
```

### Checkpoint rules

1. **Placed after each logical module**
2. **All prior tasks in module must be listed as dependencies**
3. **Explicit verification command**
4. **Include coverage check**
5. **Gate for next phase** — don't proceed if checkpoint fails

## Anti-Patterns

| ❌ Bad | ✅ Good | Why |
|--------|---------|-----|
| Write test after implementation | Test → Implement | TDD principle violated |
| Multiple behaviors in one test task | One task per behavior | Granularity rules |
| Test task without verify command | Always include verify | Agent can't confirm RED |
| Skip RED verification | Always run and confirm failure | Passing test = wrong test |
| Verify single test only | Run full test file | Misses regressions |
| Refactor while RED | Only refactor when GREEN | Tests must pass first |
| Implementation adds untested behavior | New behavior = new test first | Coverage gap |
| Checkpoint without dependencies | List all prior tasks | Ensures nothing skipped |
| No test file setup task | Create test file explicitly | Agent needs clear starting point |
| Missing error classes | Create in scaffold phase | Tests need error types |
| Hardcoded test data | Use factories | Maintainability |
| Test private methods directly | Test through public interface | Brittle tests |
| Skip flaky test | Fix root cause | Hides real bugs |
| Weak assertions (toBeDefined) | Assert specific values | False confidence |
| No commit after GREEN | Commit at each GREEN | Preserve progress |

## AI Agent Guidelines

AI agents have specific tendencies that can undermine TDD. These guidelines address common failure modes.

### Minimality Constraint (GREEN Phase)

AI agents tend to over-implement. During GREEN phase:

| Rule | Rationale |
|------|-----------|
| Write ONLY enough code to pass the failing test | TDD's design benefit comes from incremental growth |
| No error handling unless the test requires it | Error handling is a behavior—test it first |
| No edge cases unless a test covers them | Untested code is unverified code |
| No "while I'm here" improvements | Scope creep defeats the feedback loop |

**Self-check:** If implementation exceeds ~20 lines for a single test, the test may be too broad or you're over-implementing.

### One Test at a Time

Do not batch test cases before implementing.

```markdown
# ❌ Batching (AI tendency)
- [ ] Write tests for user creation
  - Test valid input succeeds
  - Test duplicate rejected
  - Test invalid email rejected
- [ ] Implement user creation  ← Implements 3 behaviors at once

# ✅ Incremental (correct TDD)
- [ ] Write test: creation succeeds with valid input
- [ ] Implement creation (minimal)
- [ ] Write test: creation rejects duplicate
- [ ] Handle duplicate check
- [ ] Write test: creation rejects invalid email
- [ ] Add email validation
```

Batching tests leads to batch implementation, which defeats TDD's design feedback loop.

### Fast Feedback Configuration

Slow test execution kills TDD efficiency. Configure for speed:

```bash
# ✅ Run single test file (fast)
npm test -- path/to/service.test.ts

# ✅ Filter to specific test (fastest)
npm test -- --grep "succeeds with valid input"

# ✅ Watch mode (immediate feedback)
npm test -- --watch

# ❌ Full suite every time (slow)
npm test
```

**Target:** < 2 seconds per RED/GREEN cycle. If tests take longer, run only the relevant file during development.

### Refactor Scope Lock

AI agents expand refactor scope beyond what's necessary. Strict boundaries:

| Allowed in REFACTOR | NOT Allowed in REFACTOR |
|---------------------|-------------------------|
| Rename variables/functions you just wrote | Modify unrelated code |
| Extract methods from code you just wrote | Add new functionality |
| Remove duplication in code you just wrote | "Improve" adjacent files |
| Simplify conditionals you just wrote | Add comments/docs to other code |
| | Refactor code from previous tasks |

**Rule:** If you didn't write it in this RED/GREEN cycle, don't touch it in REFACTOR.

### Verification Checkpoints

Before proceeding to the next test, verify:

- [ ] Current test passes
- [ ] ALL previous tests still pass (run full file)
- [ ] No unrelated changes introduced
- [ ] Implementation does ONLY what tests require

If any check fails, stop and fix before continuing.

### AI-Specific Anti-Patterns

| Anti-Pattern | Why It Happens | Mitigation |
|--------------|----------------|------------|
| Implementation before test | Training on non-TDD code | Explicit RED-first checkpoint in task |
| Over-implementing in GREEN | Training on "complete" code | Minimality constraint + line count check |
| Scope creep in REFACTOR | "While I'm here" optimization | Scope lock rule |
| Batching tests | Efficiency instinct | One-test-at-a-time rule |
| Skipping RED verification | Assuming test will fail | Must see actual failure output |
| Gaming tests | Writing code that passes test but misses intent | Review test quality before GREEN |
| Adding "helpful" extras | Tendency to be thorough | Only implement what's tested |
| Premature abstraction | Pattern recognition from training | No abstractions until duplication exists |

### Gaming Tests

AI agents may write implementation that technically passes the test but misses the intent.

```typescript
// Test
it('returns user count', async () => {
  await createUser({ name: 'Alice' })
  await createUser({ name: 'Bob' })
  const count = await service.getUserCount()
  expect(count).toBe(2)
})

// ❌ Gaming the test
getUserCount() {
  return 2  // Passes test but wrong
}

// ✅ Correct implementation
getUserCount() {
  return this.db.user.count()
}
```

**Mitigation:** After GREEN, ask: "Would this implementation work for ANY valid input, or just the test case?"

### Context Management

Long TDD sessions accumulate context. Manage it:

| Trigger | Action |
|---------|--------|
| Every 5 RED/GREEN cycles | Summarize progress in task list |
| After each checkpoint | Note completed behaviors |
| When context feels heavy | Review what's done vs remaining |
| Before complex implementation | Re-read the specific test being satisfied |

### Prompt Patterns for TDD

When instructing AI agents, use explicit TDD language:

```markdown
# ✅ Explicit TDD instruction
Write a failing test for [behavior]. Run it to confirm RED.
Then write minimal code to pass. Run all tests to confirm GREEN.

# ❌ Vague instruction
Implement [behavior] with tests.
```

The explicit version enforces the RED/GREEN sequence. The vague version allows implementation-first.
