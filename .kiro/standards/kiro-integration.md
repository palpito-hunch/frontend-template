# Kiro IDE Integration Guide for Development Standards

## Overview

This guide shows how to integrate our development standards with Kiro IDE's spec-driven development workflow. Kiro IDE uses specifications to guide AI code generation, and our standards serve as the specification foundation.

---

## Understanding Kiro IDE + Our Standards

### How Kiro IDE Works

1. You write a **specification** describing what you want
2. Kiro IDE's AI generates code based on the spec
3. Code is validated against patterns and standards
4. Iterative refinement until requirements are met

### Our Standards as Kiro Specifications

Our standards documents act as:

- **Base specifications** for all code generation
- **Validation rules** for generated code
- **Decision frameworks** for AI choices
- **Quality gates** before code acceptance

---

## Document Structure for Kiro IDE

### Primary Specifications (Always Active)

These should be loaded into Kiro's context for every session:

```
📋 Core Specifications (Load First)
├── ai-development-priority-framework.md    # Decision-making rules
├── coding-standards.md                     # SOLID, DRY, race conditions
└── when-not-to-apply-patterns.md          # Negative examples

📋 Domain Specifications (Load by Context)
├── error-message-standards.md              # When working with errors
├── file-organization-standards.md          # When creating new files
├── comment-standards.md                    # When documenting code
└── performance-standards.md                # When optimizing
```

### Workflow-Specific Checklists

Use these during specific workflows:

```
✅ Review & Validation
├── code-review-checklist.md               # Code review workflow
├── race-condition-checklist.md            # Concurrent code review
└── variable-initialization-guide.md       # Safety validation

🧪 Testing & Debugging
├── test-execution-guidelines.md           # Test running workflow
└── database-testing-fix-strategy.md       # Database test issues
```

---

## Kiro IDE Workflow Integration

### 1. Project Setup Phase

**Specification to Provide Kiro:**

```markdown
# Project: Prediction Market Backend

# Phase: Initial Setup

## Standards Context

Load these specifications:

- ai-development-priority-framework.md
- coding-standards.md
- file-organization-standards.md

## Project Structure Requirements

Follow file-organization-standards.md for:

- Directory structure (src/services/, src/repositories/, etc.)
- File naming conventions (kebab-case with type suffixes)
- Import organization patterns

## Priority Rules

Apply ai-development-priority-framework.md:

1. Financial Safety (P0)
2. Type Safety (P0)
3. SOLID Principles (P1)
4. DRY (P1)
5. Performance (P2)

## Output

Generate project structure following standards.
```

---

### 2. Feature Development Phase

**Specification Template for Kiro:**

```markdown
# Feature: [Feature Name]

# Module: [Service/Repository/Controller]

## Standards Context

Primary: coding-standards.md, ai-development-priority-framework.md
Domain: [error-message-standards.md if errors, performance-standards.md if optimizing]

## Requirements

[Your feature requirements here]

## Constraints from Standards

### MUST Follow (P0 - Critical)

- All database modifications within transactions (coding-standards.md)
- All functions have explicit return types (coding-standards.md)
- Race condition safety for financial operations (race-condition-checklist.md)
- Errors follow error-message-standards.md format

### SHOULD Follow (P1 - High Priority)

- SOLID principles for services (coding-standards.md)
- DRY for business logic (ai-development-priority-framework.md)
- File organization per file-organization-standards.md

### MAY Optimize (P2 - Lower Priority)

- Performance only if measured need (performance-standards.md)
- Caching only for expensive operations

## Decision Framework

When making trade-offs, refer to:

- ai-development-priority-framework.md "Decision Framework" section
- when-not-to-apply-patterns.md for negative examples

## Validation Checklist

Before completion, verify against:

- [ ] code-review-checklist.md (all items)
- [ ] race-condition-checklist.md (if concurrent operations)
- [ ] Zero ESLint warnings (coding-standards.md)

## Output

Generate code following all standards and constraints.
```

---

### 3. Code Review Phase

**Specification for Kiro IDE Code Review:**

```markdown
# Task: Code Review

# Target: [File/Module to Review]

## Review Specifications

Load: code-review-checklist.md, race-condition-checklist.md

## Review Process

### Step 1: SOLID Principles Review

Check against code-review-checklist.md "SOLID Principles Review" section:

- [ ] Single Responsibility
- [ ] Open/Closed
- [ ] Liskov Substitution
- [ ] Interface Segregation
- [ ] Dependency Inversion

Flag violations with specific line numbers and remediation suggestions.

### Step 2: Race Condition Safety

Check against race-condition-checklist.md:

- [ ] All database modifications within transactions
- [ ] No TOCTOU vulnerabilities
- [ ] Fresh data usage within transactions
- [ ] Concurrent operation tests present

Flag critical issues immediately.

### Step 3: Code Quality

Check against coding-standards.md:

- [ ] Variable initialization (reduce() has initial values)
- [ ] Error handling (follows error-message-standards.md)
- [ ] No code duplication (DRY)

### Step 4: Decision Validation

For each abstraction/pattern, verify against:

- when-not-to-apply-patterns.md - Should this pattern be applied?
- ai-development-priority-framework.md - Does this follow priority order?

## Output Format
```

## Review Summary

### Critical Issues (MUST FIX)

- [Issue 1 with line number and fix]
- [Issue 2 with line number and fix]

### High Priority (SHOULD FIX)

- [Issue 1 with line number and suggestion]

### Suggestions (OPTIONAL)

- [Suggestion 1]

### Approved ✅ / Request Changes ❌

```

```

---

### 4. Error Implementation Phase

**Specification for Error Handling:**

````markdown
# Task: Implement Error Handling

# Context: [Service/Feature]

## Error Specification

Load: error-message-standards.md

## Requirements for All Errors

### Structure (MUST)

```typescript
throw new BusinessLogicError(
  message, // Human-readable, includes context
  code, // UPPERCASE_SNAKE_CASE
  context, // Object with relevant data
  status // HTTP status code
);
```
````

### Message Requirements (MUST)

Every error message must include:

1. What went wrong (specific, not generic)
2. Why it went wrong (context and state)
3. Relevant data (IDs, values, current state)

### Examples from Standards

Refer to error-message-standards.md "Error Message Examples" section for:

- Market not found pattern
- Insufficient balance pattern
- Invalid input pattern
- Race condition pattern

## Validation

Before completion, verify:

- [ ] Message is actionable (tells user/dev what to do)
- [ ] Context includes all relevant IDs and values
- [ ] Error code follows naming convention
- [ ] No generic messages like "Invalid input"

## Output

Generate error handling code following all standards.

````

---

### 5. Performance Optimization Phase

**Specification for Optimization:**

```markdown
# Task: Performance Optimization
# Target: [Operation/Endpoint]

## Optimization Specification
Load: performance-standards.md

## Pre-Optimization Requirements (MUST)

### 1. Measure First
Before optimizing, provide:
- Current operation time (avg, p95, p99)
- Call frequency (per minute/hour)
- Data size (number of records, array size)

### 2. Check Always-Fix List
From performance-standards.md "ALWAYS Fix These":
- [ ] N+1 query problems?
- [ ] Database queries in loops?
- [ ] Missing indexes?
- [ ] Loading unnecessary data?
- [ ] O(n²) where O(n) possible?

If YES to any → Fix immediately, no profiling needed.
If NO to all → Continue to profiling.

### 3. Profile Decision
Check against performance-standards.md "When NOT to Optimize":
- Operation <100ms? → Don't optimize unless high frequency
- Infrequent code? → Don't optimize
- Would reduce clarity? → Profile first, measure benefit

## Optimization Approach

### If Always-Fix Issue
Apply fix from performance-standards.md examples:
- N+1 → Use include/batch query
- Loop queries → Batch query with `in`
- Missing index → Add index
- Unnecessary data → Use select
- Bad algorithm → Use better algorithm

### If Needs Profiling
1. Profile with realistic data
2. Identify actual bottleneck
3. Measure impact of proposed fix
4. Only optimize if:
   - Measurable improvement (>50% faster)
   - Doesn't significantly reduce clarity
   - Solves actual user problem

## Validation
- [ ] Performance tests added (see performance-standards.md)
- [ ] Improvement measured and documented
- [ ] Code clarity maintained or improved
- [ ] No premature optimizations introduced

## Output
Optimized code with before/after metrics and justification.
````

---

## Kiro IDE Context Files

### Create `.kiro/` Directory

Organize standards for easy loading:

```
.kiro/
├── specs/
│   ├── core/
│   │   ├── priority-framework.md          # Always load
│   │   ├── coding-standards.md            # Always load
│   │   └── when-not-to-apply.md          # Always load
│   ├── domain/
│   │   ├── errors.md                      # Load when handling errors
│   │   ├── files.md                       # Load when organizing files
│   │   ├── comments.md                    # Load when documenting
│   │   └── performance.md                 # Load when optimizing
│   └── workflows/
│       ├── code-review.md                 # Load for reviews
│       ├── race-conditions.md             # Load for concurrent code
│       └── testing.md                     # Load for tests
├── templates/
│   ├── feature-spec.md                    # Template for new features
│   ├── review-spec.md                     # Template for reviews
│   └── optimization-spec.md               # Template for optimization
└── config.yaml                            # Kiro configuration
```

### Kiro Configuration File

```yaml
# .kiro/config.yaml

# Default standards to load in every session
defaults:
  specs:
    - specs/core/priority-framework.md
    - specs/core/coding-standards.md
    - specs/core/when-not-to-apply.md

# Context-specific specs (auto-load based on file type)
contexts:
  services:
    patterns: ['src/services/**/*.ts']
    specs:
      - specs/core/coding-standards.md
      - specs/domain/errors.md

  repositories:
    patterns: ['src/repositories/**/*.ts']
    specs:
      - specs/core/coding-standards.md
      - specs/workflows/race-conditions.md

  tests:
    patterns: ['**/*.test.ts']
    specs:
      - specs/workflows/testing.md

# Validation rules (run on all generated code)
validation:
  required:
    - eslint: '--max-warnings=0'
    - typescript: 'tsc --noEmit'

  checklists:
    - specs/workflows/code-review.md
    - specs/workflows/race-conditions.md

# Performance budgets (alert if exceeded)
performance:
  budgets:
    api_endpoints:
      list: 200ms
      detail: 100ms
      write: 500ms

    database_queries:
      simple: 50ms
      joins: 100ms
      aggregations: 200ms
```

---

## Kiro IDE Prompt Templates

### Template 1: New Service Implementation

```markdown
@kiro create service

# Service: {ServiceName}

# Specifications: @specs/core/\*, @specs/domain/errors.md

## Requirements

{Your requirements here}

## Standards Constraints

- Follow SOLID principles (coding-standards.md)
- All DB operations in transactions (coding-standards.md)
- Errors follow error-message-standards.md
- No N+1 queries (performance-standards.md)

## Priority Order

1. Financial Safety → All balance changes atomic
2. Type Safety → Explicit return types
3. SOLID → Dependency injection for repositories
4. DRY → Extract common validation

## Validation

Run code-review-checklist.md before completion.

Generate service implementation.
```

### Template 2: Code Review

```markdown
@kiro review

# Target: {FilePath}

# Specifications: @specs/workflows/code-review.md, @specs/workflows/race-conditions.md

## Review Checklist

Execute all items from code-review-checklist.md:

### CRITICAL (P0)

- Race condition safety
- Financial operation atomicity
- Type safety

### HIGH (P1)

- SOLID principles
- DRY compliance
- Error standards

### MEDIUM (P2)

- Code organization
- Comment quality

## Decision Validation

For each abstraction, check:

- Should this be applied? (when-not-to-apply-patterns.md)
- Right priority? (ai-development-priority-framework.md)

## Output

Detailed review with line numbers and fix suggestions.
```

### Template 3: Fix Race Condition

```markdown
@kiro fix race-condition

# File: {FilePath}

# Issue: {Description}

# Specification: @specs/workflows/race-conditions.md

## Analysis Required

1. Identify TOCTOU vulnerabilities
2. Find stale data usage
3. Check transaction boundaries
4. Verify atomic operations

## Fix Pattern

From race-condition-checklist.md:

- Move all validation inside transaction
- Fetch fresh data in transaction
- Use batch operations for related updates
- Add concurrent operation tests

## Validation

- [ ] All checks inside transaction
- [ ] No stale data usage
- [ ] Tests for concurrent scenarios

Generate fixed code with concurrent tests.
```

### Template 4: Optimize Performance

```markdown
@kiro optimize

# Target: {Operation}

# Current: {CurrentMetrics}

# Specification: @specs/domain/performance.md

## Step 1: Check Always-Fix List

- [ ] N+1 queries?
- [ ] Queries in loops?
- [ ] Missing indexes?
- [ ] Unnecessary data loaded?

## Step 2: Profile (if not always-fix)

Measure with realistic data:

- Operation time: {time}
- Call frequency: {frequency}
- Data size: {size}

## Step 3: Optimize Only If

- Measurable bottleneck (>100ms + frequent)
- Clear improvement possible
- Maintains code clarity

## Output

Optimized code + before/after metrics + justification.
```

---

## Quick Reference Card for Kiro

Save this as `.kiro/quick-reference.md`:

```markdown
# Kiro IDE Quick Reference

## Load These First (Every Session)

@specs/core/priority-framework.md
@specs/core/coding-standards.md
@specs/core/when-not-to-apply.md

## Priority Order (When in Conflict)

1. Financial Safety (P0) → Transactions, race conditions
2. Type Safety (P0) → Explicit types, null handling
3. SOLID (P1) → Clean architecture
4. DRY (P1) → No duplication
5. Performance (P2) → Optimize only when measured

## Common Commands

### New Service

@kiro create service --spec=feature-spec.md

### Code Review

@kiro review --spec=code-review.md

### Fix Race Condition

@kiro fix --spec=race-conditions.md

### Optimize

@kiro optimize --spec=performance.md --profile-first

## Quick Checks

### Before Committing

- [ ] ESLint: 0 warnings
- [ ] All reduce() have initial values
- [ ] Transactions for financial operations
- [ ] Errors follow standards

### Red Flags

❌ Database write outside transaction
❌ reduce() without initial value
❌ Generic error messages
❌ Missing return types

## When NOT to Apply

- DRY: Different concepts, only 2 uses
- DI: Pure functions, utilities
- Transactions: Read-only operations
- Optimization: <100ms, infrequent, unclear benefit

## Standards Docs Path

All specs in: .kiro/specs/
```

---

## Integration Workflow Example

### Complete Feature Development Flow

```markdown
# Kiro Session: Implement Market Trading Feature

## Session Setup

@kiro start --load-defaults
@specs/domain/errors.md
@specs/workflows/race-conditions.md

## Step 1: Generate Service

@kiro create service --name=MarketTradingService

Spec:

- Follow coding-standards.md
- All trade operations in transactions
- Race condition safe (race-condition-checklist.md)
- Errors follow error-message-standards.md

[Kiro generates code]

## Step 2: Review Generated Code

@kiro review --file=market-trading.service.ts

Check:

- code-review-checklist.md (all items)
- race-condition-checklist.md (transaction safety)
- when-not-to-apply-patterns.md (no over-engineering)

[Kiro identifies issues]

## Step 3: Fix Issues

@kiro fix --issues=[list from review]

[Kiro applies fixes]

## Step 4: Add Tests

@kiro generate tests --file=market-trading.service.ts

Requirements:

- Unit tests for business logic
- Race condition tests (from race-condition-checklist.md)
- Property-based tests for financial calculations

[Kiro generates tests]

## Step 5: Final Validation

@kiro validate --strict

Checks:

- ESLint: 0 warnings
- TypeScript: no errors
- Tests: all passing
- Standards: full compliance

[Kiro confirms all pass]

## Step 6: Generate Documentation

@kiro document --file=market-trading.service.ts

Follow: comment-standards.md

- JSDoc for public methods
- Complex algorithm explanations
- Race condition prevention notes

[Kiro adds documentation]

## Complete ✅
```

---

## Summary: Kiro IDE + Standards Integration

### Key Principles

1. **Standards as Specifications**: Your standards docs are the spec foundation
2. **Context-Aware Loading**: Load relevant standards for each task
3. **Priority-Driven**: Use ai-development-priority-framework.md for conflicts
4. **Validation Gates**: Check against checklists before completion
5. **Iterative Refinement**: Review → Fix → Validate cycle

### Best Practices

✅ **DO:**

- Load core specs in every session
- Reference specific standard sections in prompts
- Use templates for consistency
- Validate against checklists before commit
- Keep standards docs updated

❌ **DON'T:**

- Assume Kiro knows standards without loading them
- Skip validation steps
- Override safety standards for simplicity
- Generate code without spec context

### Benefits

- **Consistency**: All code follows same standards
- **Quality**: Built-in validation against best practices
- **Speed**: Templates accelerate development
- **Safety**: Race conditions caught automatically
- **Maintainability**: Standards enforced from generation

---

**Remember**: Kiro IDE is most effective when given clear specifications. Your standards documents ARE those specifications. Load them, reference them, validate against them.
