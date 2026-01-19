# Tech Stack

This repository contains reference configurations. Projects using these standards may have different stacks.

## Reference Configurations

- **TypeScript 5** - Reference tsconfig.json with strict settings
- **ESLint 9** - Reference eslint.config.mjs with TypeScript rules
- **Prettier 3** - Code formatting
- **Husky** - Git hooks for pre-commit checks
- **commitlint** - Conventional commit enforcement
- **semantic-release** - Automated versioning

## Standards Coverage

The standards in this repository cover:

- **TypeScript** - Style and architecture patterns
- **Prisma** - Database operations and transactions
- **Next.js** - App Router, Server/Client Components
- **Zod** - Schema validation patterns

## Common Commands

```bash
npm run lint         # Check for linting errors
npm run lint:fix     # Auto-fix linting errors
npm run format       # Format code with Prettier
npm run format:check # Check formatting
npm run type-check   # TypeScript type checking
npm run validate     # Run all checks
```
