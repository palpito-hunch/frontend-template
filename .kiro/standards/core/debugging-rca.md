# Debugging & Root Cause Analysis

## Purpose

This document defines a structured approach for debugging issues and performing root cause analysis. It ensures alignment on methodology, systematic investigation, and expected deliverables when resolving problems.

---

## When to Apply

- Investigating bugs or unexpected behavior
- Troubleshooting production incidents
- Analyzing test failures
- Diagnosing performance issues
- Any situation where the root cause is not immediately obvious

## When NOT to Apply

- Obvious typos or syntax errors with clear fixes
- Well-understood issues with documented solutions
- Simple configuration changes with predictable outcomes

---

## Approach

### Phase 1: Before Starting

1. **Clarify the symptom** - What's the expected vs actual behavior?
2. **Gather context** - When did it start? What changed? Who's affected?
3. **Reproduce first** - Confirm you can trigger the issue reliably

> Do not proceed to fixes until you can reproduce the issue or have strong evidence of the cause.

### Phase 2: Investigation

1. **Form a hypothesis** - State what you think is wrong and why
2. **Test systematically** - One variable at a time, don't shotgun changes
3. **Follow the data** - Logs, traces, state inspection—not guesses
4. **Document dead ends** - Note what was ruled out and why

### Phase 3: Resolution

1. **Identify root cause** - Not just the symptom, the underlying why
2. **Propose fix** - Explain the change and why it addresses the root cause
3. **Verify** - Confirm the fix works and doesn't break other things

---

## Expected Artifacts

After completing debugging/RCA, produce these artifacts:

| Artifact | Description | Required |
|----------|-------------|----------|
| Root cause summary | 1-2 sentences explaining the underlying issue | Yes |
| Fix with explanation | The code change and why it addresses the root cause | Yes |
| Reproduction test | Test that would catch this issue (when feasible) | Recommended |
| Follow-up items | Monitoring, related issues, tech debt identified | If applicable |

### Root Cause Summary Format

```
**Root Cause:** [Brief description of what was actually wrong]
**Why it happened:** [The underlying reason this occurred]
**Fix:** [What was changed to resolve it]
```

---

## Principles

### Don't Guess

If unsure, investigate more before changing code. Random changes waste time and can introduce new bugs.

```
❌ "Let me try changing this and see if it helps"
✅ "Based on the logs showing X, I believe the issue is Y because Z"
```

### Smallest Fix First

Address the root cause directly. Don't refactor the world while fixing a bug.

```
❌ Fixing a null pointer by also restructuring the entire module
✅ Fixing the null pointer, then creating a separate ticket for refactoring
```

### Ask Before Assuming

Clarify unknowns rather than filling gaps with assumptions.

```
❌ "I assume this service always returns a valid response"
✅ "I need to verify: does this service ever return null or error?"
```

---

## Examples

### Good Investigation Flow

```
1. User reports: "Orders are failing intermittently"
2. Gather context: Started 2 days ago, affects ~5% of orders
3. Reproduce: Found pattern - fails when item quantity > 100
4. Hypothesis: Integer overflow in quantity calculation
5. Test: Check calculation logic, find it uses Int16
6. Verify: Confirmed Int16.MaxValue = 32,767, not 100
7. New hypothesis: Database constraint on quantity field
8. Test: Check schema, find CHECK constraint quantity <= 100
9. Root cause: Database constraint added in recent migration
10. Fix: Update constraint or validate in application layer
```

### Bad Investigation Flow

```
1. User reports: "Orders are failing intermittently"
2. "Let me restart the server and see if that helps"
3. Still failing
4. "Let me increase timeouts"
5. Still failing
6. "Let me add more logging everywhere"
7. ...hours later, still no root cause
```

---

## Checklist

### Before Investigation

- [ ] Can clearly state the expected vs actual behavior
- [ ] Know when the issue started or was first noticed
- [ ] Can reproduce the issue (or have strong evidence if not reproducible)

### During Investigation

- [ ] Have a current hypothesis written down
- [ ] Testing one variable at a time
- [ ] Documenting what's been ruled out

### Before Closing

- [ ] Root cause identified and documented
- [ ] Fix addresses root cause (not just symptom)
- [ ] Verified fix works
- [ ] Reproduction test added (if feasible)
- [ ] Follow-up items captured (if any)
- [ ] **Prevention rule added to ai-rules repository** (see below)

---

## Capturing Learnings (Mandatory)

**The RCA process is not complete until a rule has been added to the ai-rules repository that addresses the issue.**

Every debugging session should result in either:

1. **A new rule** - If this type of issue isn't covered by existing standards
2. **An updated rule** - If existing standards need clarification or strengthening
3. **A red flag entry** - If this is a pattern that should be auto-rejected

### Why This Matters

- Prevents the same class of bug from recurring
- Builds organizational knowledge over time
- Ensures AI assistants learn from past mistakes
- Creates a feedback loop between debugging and prevention

### How to Capture

```bash
# 1. Identify the prevention rule
# Ask: "What rule, if followed, would have prevented this bug?"

# 2. Determine where it belongs
# - Red Flags list in CLAUDE.*.md (if it's a "never do this" pattern)
# - Existing standard file (if it extends current guidance)
# - New standard file (if it's a new category)

# 3. Create PR to ai-rules repository
git checkout -b fix/add-rule-for-[issue-type]
# Make changes
gh pr create --title "fix: add rule to prevent [issue type]"
```

### Examples

| Bug Found | Rule Added |
|-----------|------------|
| Auth token stored in localStorage | Red flag: "Storing auth tokens in localStorage/sessionStorage" |
| N+1 query in user list | Red flag: "N+1 query patterns" |
| Missing null check crashed app | Standard update: Added null handling examples to typescript/style.md |
| Race condition in checkout | New section: "Race Conditions" in quick-reference.md |

---

## Integration with Other Standards

- **Error Handling** (`domain/errors.md`): Use specific error classes to make debugging easier
- **Testing** (`domain/testing-mocks.md`): Write regression tests for bugs found
- **Performance** (`domain/performance.md`): Profile before optimizing performance issues
- **Git Workflow** (`domain/git-workflow.md`): Bug fixes go through normal PR process
