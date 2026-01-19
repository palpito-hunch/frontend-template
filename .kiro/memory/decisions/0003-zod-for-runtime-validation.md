# ADR-0003: Zod for Runtime Validation

**Date**: 2024-01
**Status**: Accepted

## Context

Need runtime validation because:
- TypeScript types are erased at runtime
- Form data needs validation
- API responses need validation
- Environment variables need validation

## Decision

Use Zod for all runtime validation and type inference.

## Consequences

**Positive:**
- Single source of truth for types and validation
- Excellent TypeScript integration with `z.infer`
- Works well with React Hook Form
- Good error messages for forms

**Negative:**
- Another dependency to learn
- Some overlap with TypeScript types

## References

- `standards/libraries/zod.md`
