# Feature: [Feature Name]

> **Status**: Draft | In Review | Approved | Implemented
> **Author**: [Name]
> **Created**: [Date]
> **Updated**: [Date]

## Intent

<!-- What this feature should accomplish (the "why"). Be specific about the problem being solved. -->

[Describe the purpose and motivation for this feature]

---

## User Stories

<!-- Who is this for and what do they need? -->

### Primary User Story

As a [type of user],
I want [goal/desire],
so that [benefit/reason].

### Additional Stories (if applicable)

- As a [user], I want [goal] so that [benefit].
- As a [user], I want [goal] so that [benefit].

---

## Behavior

<!-- Define expected behavior using Given/When/Then format -->

### Scenario 1: [Happy Path Name]

```gherkin
Given [initial context/state]
When [action is performed]
Then [expected outcome]
And [additional outcome if applicable]
```

### Scenario 2: [Edge Case Name]

```gherkin
Given [initial context/state]
When [action is performed]
Then [expected outcome]
```

### Scenario 3: [Error Case Name]

```gherkin
Given [initial context/state]
When [invalid action is performed]
Then [error handling behavior]
```

---

## Technical Approach

<!-- High-level approach, not implementation details -->

### Components Affected

- [ ] `src/services/[service].ts` - [What changes]
- [ ] `src/repositories/[repo].ts` - [What changes]
- [ ] `src/pages/api/[route].ts` - [What changes]

### Data Model Changes

<!-- Any database schema changes required -->

```prisma
// Example schema changes (if applicable)
model Example {
  id        String   @id @default(uuid())
  // new fields...
}
```

### API Changes

<!-- New or modified endpoints -->

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/[resource]` | [Description] |
| GET | `/api/[resource]/:id` | [Description] |

---

## Constraints

<!-- Standards and rules that must be followed -->

### Must Follow

- [ ] `core/priority-framework.md` - Priority P0/P1 rules
- [ ] `typescript/architecture.md` - Layered architecture
- [ ] `typescript/style.md` - ESLint/TypeScript rules
- [ ] `libraries/prisma.md` - Transaction requirements (if DB writes)
- [ ] `domain/errors.md` - Error handling standards

### Security Considerations

- [ ] Input validation using Zod schemas
- [ ] Authentication/authorization checks
- [ ] No sensitive data in logs
- [ ] SQL injection prevention (use Prisma)

### Performance Considerations

- [ ] No N+1 queries
- [ ] Pagination for list endpoints
- [ ] Appropriate caching strategy

---

## Acceptance Criteria

<!-- Definition of done - these become test cases -->

### Functional

- [ ] [Criterion 1 - specific and testable]
- [ ] [Criterion 2 - specific and testable]
- [ ] [Criterion 3 - specific and testable]

### Non-Functional

- [ ] All tests pass
- [ ] ESLint: 0 warnings
- [ ] TypeScript: no errors
- [ ] API response time < 200ms
- [ ] Code reviewed and approved

---

## Anti-Patterns

<!-- Specific things to avoid in this implementation -->

- **Do NOT** [specific thing to avoid and why]
- **Do NOT** [specific thing to avoid and why]
- **Do NOT** [specific thing to avoid and why]

---

## Test Plan

<!-- How this feature will be tested -->

### Unit Tests

- [ ] [Service method] - [what it tests]
- [ ] [Service method] - [what it tests]

### Integration Tests

- [ ] [API endpoint] - happy path
- [ ] [API endpoint] - error cases

### Edge Cases to Test

- [ ] [Edge case 1]
- [ ] [Edge case 2]

---

## Open Questions

<!-- Anything that needs clarification before implementation -->

1. [Question 1]
2. [Question 2]

---

## References

<!-- Related documents, tickets, or external resources -->

- [Link to related ticket/issue]
- [Link to design document]
- [Link to relevant standard]

---

## Changelog

| Date | Author | Change |
|------|--------|--------|
| [Date] | [Name] | Initial draft |
