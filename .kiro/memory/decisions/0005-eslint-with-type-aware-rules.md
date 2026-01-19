# ADR-0005: ESLint with Type-Aware Rules

**Date**: 2024-01
**Status**: Accepted

## Context

Need linting that:
- Catches common errors
- Enforces coding standards
- Works with TypeScript and React

## Decision

Use ESLint with Next.js recommended config and type-aware rules.

## Consequences

**Positive:**
- Catches React-specific issues
- Enforces accessibility best practices
- Consistent code style
- Integrates with Next.js

**Negative:**
- Slower linting (requires type information)
- More configuration complexity

## References

- `eslint.config.mjs`
- `standards/typescript/style.md`
