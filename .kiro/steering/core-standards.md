---
inclusion: always
---

# Core Coding Standards

These standards apply to ALL code generation. They define the priority framework, key principles, and anti-patterns.

## Quick Reference

#[[file:.kiro/standards/quick-reference.md]]

## Priority Framework

When standards conflict, use this framework to decide:

#[[file:.kiro/standards/core/priority-framework.md]]

## When NOT to Apply Patterns

Before adding abstractions, check if the pattern should be applied:

#[[file:.kiro/standards/core/when-not-to-apply.md]]

## AI Behavior Guidelines

How AI should interact with this codebase:

#[[file:.kiro/standards/core/ai-behavior.md]]

## Product Development Workflow (MANDATORY)

**CRITICAL: Product development has six phases, executed in order.**

**All workflow rules are the SINGLE SOURCE OF TRUTH in `.kiro/standards/core/`:**

### Phase 0: Product Brief Creation (FOUNDATION) — Owner: PM

- PM identifies problem/requirement/opportunity
- PM drafts brief with AI assistance
- PM curates extensively for clarity and completeness
- Quality gate: Brief must be approved before proceeding

**Key Principle:** AI-assisted, PM-curated. Depth over speed. No ambiguity.

### Phase 1: Product Brief → Projects — Owner: PM + AI

#[[file:.kiro/standards/core/linear-mcp-product-to-projects.md]]

**Use after a product brief is approved to break it down into sprint-sized Linear projects.**

### Phase 2: Spec Creation / Feature Refinement — Owner: PM + Engineering

- Collaborative refinement session (like super-charged Scrum refinement)
- Create specs (requirements.md, design.md, tasks.md) with AI assistance
- PM brings product context, Engineering brings technical feasibility
- **AI MUST use `AskUserQuestion` tool extensively** to probe edge cases, clarify ambiguities, explore trade-offs, and ensure completeness
- Quality gate: PM and Engineering sign off before proceeding

**Key Principle:** Collaborative, AI-assisted, thorough. Deep questioning produces better specs.

### Phase 3: Spec-to-Project — Owner: AI

#[[file:.kiro/standards/core/linear-mcp-spec-to-project.md]]

**Use after specs are approved to populate the Linear project with issues.**

### Phase 4: Task Development (MANDATORY FOR ALL AGENTS) — Owner: AI

#[[file:.kiro/standards/core/linear-mcp-task-development.md]]

**This workflow is non-negotiable and applies to ALL task development.**

### Phase 5: Feature Verification — Owner: PM + Engineering

- PM reviews completed feature against requirements
- Engineering demonstrates implementation
- Verify all acceptance criteria are met
- Quality gate: Sign off before PR review begins

**Key Principle:** Validate feature completeness before formal code review.

### Overview

#[[file:.kiro/standards/core/linear-mcp-rules.md]]

**Violation of the task development workflow constitutes incorrect agent behavior.**
