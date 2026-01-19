# Project Structure

## Repository Organization

```
├── .kiro/                      # AI rules and configuration
│   ├── steering/               # Kiro steering files (this directory)
│   ├── standards/              # Coding standards (referenced by steering)
│   ├── memory/                 # ADRs and glossary
│   ├── templates/              # Spec templates
│   ├── validation/             # Machine-readable validation rules
│   └── specs/                  # Feature specifications (Kiro-generated)
├── CLAUDE.md                   # Entry point for Claude tools
├── README.md                   # Repository documentation
├── tsconfig.json               # Reference TypeScript config
└── eslint.config.mjs           # Reference ESLint config
```

## Standards Directory (`standards/`)

```
standards/
├── core/                       # Always-loaded standards
│   ├── priority-framework.md   # P0-P3 decision rules
│   ├── ai-behavior.md          # AI interaction guidelines
│   └── when-not-to-apply.md    # Anti-patterns
├── typescript/                 # TypeScript-specific
│   ├── style.md                # Code style rules
│   └── architecture.md         # Structural patterns
├── libraries/                  # Library-specific
│   ├── prisma.md               # Database operations
│   ├── nextjs.md               # Next.js patterns
│   └── zod.md                  # Schema validation
├── domain/                     # Domain-specific
│   ├── errors.md               # Error handling
│   ├── performance.md          # Optimization rules
│   ├── testing-mocks.md        # Mock policy
│   └── git-workflow.md         # Git conventions
└── quick-reference.md          # One-page summary
```

## How Standards Are Used

1. **Claude** reads `CLAUDE.md` which points to `standards/`
2. **Kiro** reads steering files which reference `standards/` via `#[[file:]]`
3. Both AIs use the same underlying standards
