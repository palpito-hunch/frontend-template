# AI Development Standards

This project uses structured standards for AI-assisted code generation. Both Claude (Claude Code, Cursor, Claude.ai) and Kiro IDE should follow these standards.

## Standards Location

All standards are in `.kiro/standards/`. Start with the quick reference:

```
.kiro/standards/quick-reference.md
```

## Priority Hierarchy (P0 > P1 > P2 > P3)

When standards conflict, higher priority wins:

- **P0 (Critical):** Financial safety, Type safety
- **P1 (High):** SOLID principles, DRY
- **P2 (Medium):** Performance (measure first, then optimize)
- **P3 (Low):** Code brevity

Full framework: `.kiro/standards/core/priority-framework.md`

## Red Flags (Never Generate)

- `reduce()` without initial value
- Generic `Error` class instead of specific error types
- Missing return type annotations on public functions
- Floating promises (unhandled async)
- Using `any` type instead of proper TypeScript types
- Missing key prop in React list rendering

## Key Standards

| Category   | File                                         | When to Use                |
| ---------- | -------------------------------------------- | -------------------------- |
| Quick Ref  | `.kiro/standards/quick-reference.md`         | Always                     |
| Priorities | `.kiro/standards/core/priority-framework.md` | Resolving conflicts        |
| Anti-patterns | `.kiro/standards/core/when-not-to-apply.md` | Before adding abstractions |
| TypeScript | `.kiro/standards/typescript/style.md`        | Writing TypeScript         |
| Errors     | `.kiro/standards/domain/errors.md`           | Error handling             |
| Testing    | `.kiro/standards/domain/testing-mocks.md`    | Writing tests              |

## Library Standards

| Library | File                                    | When to Use            |
| ------- | --------------------------------------- | ---------------------- |
| Next.js | `.kiro/standards/libraries/nextjs.md`   | Next.js features       |
| Zod     | `.kiro/standards/libraries/zod.md`      | Schema validation      |

## Validation Rules

Machine-readable patterns in `.kiro/validation/rules.yml` define what to check during code generation.

## Project Memory

- **ADRs**: `.kiro/memory/decisions/` - Why architectural choices were made
- **Glossary**: `.kiro/memory/glossary.yml` - Domain terminology

## ADR Compliance (Mandatory)

**ALWAYS check ADRs before making architectural changes.** If a proposed change contradicts an existing ADR:

1. **STOP** - Do not proceed with the change
2. **NOTIFY** - Inform the user that the change conflicts with ADR-XXXX
3. **ASK** - Request explicit approval to either:
   - Modify the approach to comply with the ADR
   - Update/supersede the ADR (requires justification)

ADRs represent deliberate architectural decisions. Contradicting them without review undermines the decision-making process.

## Documentation Updates (Mandatory)

**ALWAYS keep documentation in sync with code changes.** After completing any task:

1. **Review impacted documentation**: README, ADRs, inline comments, API docs
2. **Update immediately**: Don't defer documentation updates to "later"
3. **Verify accuracy**: Ensure examples, diagrams, and references are still correct
4. **Add new docs**: Document new features, APIs, or architectural decisions

Common documentation touchpoints:
- **README.md**: Features, setup instructions, architecture diagrams
- **ADRs** (`.kiro/memory/decisions/`): Architectural decisions and their rationale
- **API documentation**: Endpoint changes, request/response formats
- **Code comments**: Complex logic, non-obvious decisions
- **CHANGELOG.md**: User-facing changes

**Outdated documentation is worse than no documentation.** If you change code, update the docs in the same PR.

## Conventions

- **Files:** kebab-case (`user-profile.tsx`)
- **Components:** PascalCase (`UserProfile.tsx`)
- **Functions/Variables:** camelCase
- **Constants:** UPPER_SNAKE_CASE
- **Commits:** Conventional commits format

Full conventions: `.kiro/conventions.yml`

## Working with Kiro IDE

When this project uses Kiro IDE for spec-driven development:

1. **Kiro creates specs** in `.kiro/specs/features/` using the template at `.kiro/templates/feature-spec.md`
2. **Both AIs reference the same standards** in `.kiro/standards/`
3. **Validation rules** in `.kiro/validation/rules.yml` apply to all generated code

Claude should follow specs created by Kiro, and Kiro should follow the same standards Claude uses.

## Project Configuration

See `.kiro/steering.yml` for project-specific settings (stack, preferences).

## Key Principles

1. **Default to safety** - More validations, explicit error handling
2. **Fail explicitly** - Descriptive errors with context, never silent failures
3. **One pattern at a time** - Don't combine patterns unnecessarily
4. **Measure before optimizing** - No premature optimization

## Creating New Projects (Mandatory)

**ALWAYS use GitHub's template feature** when creating projects from organization templates:

```bash
# Correct - uses GitHub template feature
gh repo create org/new-project --template palpito-hunch/frontend-template --private --clone

# Wrong - cloning loses template attribution and carries git history
git clone org/frontend-template new-project  # DON'T DO THIS
```

This project was created from: `palpito-hunch/frontend-template`

Benefits of `--template`:
- Clean git history (starts fresh)
- GitHub shows "generated from" attribution
- Proper template repository relationship

## Git Workflow (Mandatory)

**NEVER commit directly to main.** All changes must go through a feature branch and PR:

1. Create a feature branch: `git checkout -b feature/description`
2. Make changes and commit to the branch
3. Push and create a PR: `gh pr create`
4. Merge via PR after approval: `gh pr merge --squash --delete-branch`

No exceptions, even for small changes.
