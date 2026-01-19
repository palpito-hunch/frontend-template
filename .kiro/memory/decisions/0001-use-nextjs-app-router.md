# ADR-0001: Use Next.js App Router

**Date**: 2024-01
**Status**: Accepted

## Context

Need a React framework that supports:
- Server-side rendering for SEO
- API routes for backend functionality
- Modern React patterns (Server Components)
- Good developer experience

## Decision

Use Next.js 16+ with App Router (not Pages Router).

## Consequences

**Positive:**
- Server Components reduce client bundle size
- React Compiler provides automatic memoization
- Built-in support for streaming and Suspense
- Excellent TypeScript support
- Turbopack for fast development builds

**Negative:**
- Learning curve for Server vs Client Components
- Some libraries not yet compatible with App Router
- Different mental model from traditional React

## References

- `src/app/` - App Router directory
- `standards/libraries/nextjs.md`
