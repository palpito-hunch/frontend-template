# ADR-0002: Use Tailwind CSS

**Date**: 2025-01
**Status**: Accepted

## Context

Need a styling solution that:
- Enables rapid UI development
- Provides consistent design system
- Works well with component-based architecture
- Has good performance characteristics

## Decision

Use Tailwind CSS v4 with CSS-first configuration.

## Consequences

**Positive:**
- Utility-first approach speeds up development
- No context switching between files
- Automatic content detection (no config needed)
- 3-10x faster builds than v3
- Built-in design system with customization

**Negative:**
- Learning curve for utility classes
- HTML can become verbose
- Team needs to agree on patterns

## References

- `src/app/globals.css` - Tailwind import
- `postcss.config.mjs` - PostCSS configuration
