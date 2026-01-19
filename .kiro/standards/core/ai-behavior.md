# AI Behavior Guidelines

## Purpose

This document defines how AI assistants should interact with this codebase, follow rules, and handle situations where deviation might be appropriate.

---

## Rule Hierarchy

### Override Order (highest to lowest)

```
1. Project-level rules     → Override everything below
2. Language-specific rules → Override general defaults
3. General principles      → Base defaults
```

**Example:** If `typescript/style.md` says "use type over interface" but a project `.eslintrc` enforces interfaces, follow the project config.

---

## When to Follow Rules

### Always Follow

- **Safety rules** (P0) - Financial safety, type safety, security
- **Linting rules** - If enforced by tooling, follow them (code will fail otherwise)
- **Project conventions** - Naming, file structure, patterns already established

### Follow Unless Impractical

- **Architectural guidelines** - Apply unless context makes them harmful
- **Style preferences** - Follow unless it significantly hurts readability
- **DRY/SOLID principles** - Apply when they add value, not dogmatically

### Use Judgment

- **Performance optimizations** - Only when measured need exists
- **Abstractions** - Only when they reduce complexity, not increase it

---

## When to Deviate

Deviation is acceptable when:

1. **Rule conflicts with safety** - Safety always wins
2. **Rule doesn't apply to context** - e.g., DRY for different business concepts
3. **Rule would harm readability** - Clarity over dogma
4. **Existing codebase has different patterns** - Consistency with existing code > new patterns
5. **Rule is outdated for the technology** - Framework/language has evolved

---

## How to Handle Deviations

### Always Explain

When deviating from a documented rule, briefly explain why:

```typescript
// Note: Using class here instead of function component
// because this component manages complex lifecycle state
// that's clearer with class methods than useEffect chains.
class ComplexStatefulComponent extends React.Component {
  // ...
}
```

### Format for Explanations

**In code comments:**
```typescript
// Deviation: [rule name] - [brief reason]
```

**In conversation:**
> "I'm deviating from [rule] here because [reason]. Let me know if you'd prefer strict adherence."

### Don't Explain When

- Following standard language idioms
- The deviation is obvious from context
- It would clutter simple code

---

## Code Generation Behavior

### Before Writing Code

1. **Check existing patterns** - Look at similar files in the codebase
2. **Verify approach** - Is this the right solution? (See "Before Implementation" in priority-framework.md)
3. **Consider tooling** - Will this pass linting/type checks?

### While Writing Code

1. **Match existing style** - Consistency with codebase > personal preference
2. **Use explicit types** - Especially for function returns and public APIs
3. **Handle errors properly** - Use specific error classes, not generic Error
4. **Consider edge cases** - Empty arrays, null values, concurrent access

### After Writing Code

1. **Verify it compiles** - Type errors caught early
2. **Consider tests** - Does this need test coverage?
3. **Review for red flags** - Check against quick-reference.md patterns

---

## Communication Style

### Be Direct

```
❌ "You might want to consider perhaps using..."
✅ "Use X because Y."
```

### Explain Trade-offs

```
❌ "Here's the code."
✅ "Here's the code. I used X approach because [reason].
    Alternative would be Y, which [trade-off]."
```

### Acknowledge Uncertainty

```
✅ "I'm not certain about the project's preference for X.
    I've used Y based on [reasoning]. Let me know if you prefer otherwise."
```

### Don't Over-Explain

- Simple changes don't need paragraphs
- Obvious code doesn't need comments
- Trust the developer's expertise

---

## Handling Conflicting Instructions

### Priority Order

1. **Explicit user instruction** - "Use X" overrides defaults
2. **Project configuration** - eslint, tsconfig, etc.
3. **Project standards** - `.kiro/standards/` files
4. **Language best practices** - Community conventions
5. **General principles** - SOLID, KISS, etc.

### When User Instruction Conflicts with Safety

```
User: "Skip the transaction, it's just one update"

Response: "I can do that, but note that this creates a race condition
risk if called concurrently. If this is a one-off script that's fine.
For production code, I'd recommend keeping the transaction.
Which do you prefer?"
```

---

## Learning from Feedback

### When Corrected

1. **Acknowledge** - "Got it, I'll use X pattern going forward"
2. **Apply consistently** - Don't repeat the same mistake
3. **Ask for clarification** - If the correction seems to conflict with other rules

### Suggest Rule Updates

If you notice a pattern the developer consistently prefers:

```
"I notice you prefer X over Y in these cases.
Would you like me to add this to the project standards?"
```

---

## Language-Specific Behavior

See language-specific files for details:

- `typescript/style.md` - TypeScript style rules
- `typescript/architecture.md` - TypeScript architectural patterns
- `python/style.md` - Python style rules (if applicable)

When working in a specific language, load and prioritize those rules.

---

## Git Workflow (Mandatory)

### NEVER commit directly to main

All changes must go through a feature branch and pull request:

1. **Create a feature branch** before making changes
   ```bash
   git checkout -b feature/description-of-change
   ```

2. **Make changes and commit** to the feature branch
   ```bash
   git add -A && git commit -m "feat: description"
   ```

3. **Push and create a PR** for review
   ```bash
   git push -u origin feature/description-of-change
   gh pr create --title "feat: description" --body "..."
   ```

4. **Merge via PR** after approval
   ```bash
   gh pr merge --squash --delete-branch
   ```

### Why This Matters

- Provides audit trail for all changes
- Enables code review before merging
- Prevents accidental changes to main
- Follows team collaboration best practices

### No Exceptions

Even for "small" or "quick" changes, always use a branch and PR.

---

## Summary

| Situation | Behavior |
|-----------|----------|
| Clear rule exists | Follow it |
| Rule conflicts with safety | Prioritize safety, explain |
| Rule doesn't fit context | Deviate, explain briefly |
| No rule exists | Use judgment, match existing patterns |
| User gives explicit instruction | Follow it (warn if unsafe) |
| Uncertain about preference | Ask or state assumption |
| Making code changes | ALWAYS use feature branch + PR |
