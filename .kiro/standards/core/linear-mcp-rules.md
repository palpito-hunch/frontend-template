# Product Development Workflow Rules
---
inclusion: always
---

## Purpose

This document provides an overview of the product development workflow and Linear MCP integration rules.

The product development workflow has six phases, executed in order:

0. **Product Brief Creation** - PM creates a well-defined product brief (foundation)
1. **Product Brief → Projects** - Breaking down the brief into Linear projects
2. **Spec Creation / Feature Refinement** - PM + Engineering create specs with AI assistance
3. **Spec-to-Project** - AI populates project with issues from approved specs
4. **Task Development** - AI develops tasks with proper status tracking
5. **Feature Verification** - PM + Engineering verify feature before PR review

**Key Concept**: Problem → Product Brief → Projects → Specs → Issues → Development → Verification → PR Review.

---

## Quick Reference

### Step 0: Product Brief Creation (FOUNDATION)

**When to use:** When a PM identifies a problem, requirement, or opportunity to address.

**Process:**
1. PM identifies a problem, requirement, or opportunity
2. PM drafts initial product brief
3. PM collaborates with AI to refine and improve
4. PM curates extensively for clarity and completeness
5. Brief is reviewed and approved

**Key principles:**
- AI-assisted, PM-curated (PM owns the final document)
- Depth over speed (take time to create a solid brief)
- Clarity is paramount (no ambiguity)
- Quality gate: Brief must be complete before Step 1

---

### Step 1: Breaking Down Product Briefs into Projects

See: `.kiro/standards/core/linear-mcp-product-to-projects.md`

**When to use:** After a product brief is created and approved.

**Key rules:**
- Analyze the product brief to identify features
- Break down into sprint-sized projects (1-2 weeks each)
- Present projects for PM review and approval
- Priority: 2 (High), 3 (Medium), 4 (Low) — never 1 (Urgent)
- Names: Title Case, no emojis (use icons)
- Manual follow-up: Initiative linking, labels in Linear UI

---

### Step 2: Spec Creation / Feature Refinement

**When to use:** After a project exists in Linear (Step 1 complete), before creating issues.

**Owners:** Product Manager + Engineering (AI-assisted)

**Process:**
1. Review the Linear project definition
2. PM and Engineering collaborate to define requirements
3. Use AI to draft and refine specification documents
4. Create specs: `requirements.md`, `design.md`, `tasks.md`
5. Review, iterate, resolve ambiguities
6. PM and Engineering sign off

**Spec File Locations:**
```
.kiro/specs/[feature]/
├── requirements.md    # What must be true
├── design.md          # How correctness is achieved
└── tasks.md           # How work is executed
```

**Key Principles:**
- **Collaborative**: PM (product context) + Engineering (technical feasibility)
- **AI-assisted**: AI drafts and identifies gaps—humans curate and approve
- **Thorough**: Resolve ambiguities before development starts
- **Quality gate**: Both PM and Engineering must sign off before Step 3

**IMPORTANT:** AI agents MUST use `AskUserQuestion` tool extensively to:
- Probe for edge cases and failure modes
- Clarify ambiguous requirements
- Explore trade-offs and alternatives
- Ensure completeness of acceptance criteria

Deep questioning produces better specs and reduces rework.

---

### Step 3: Populating Projects with Issues from Specs

See: `.kiro/standards/core/linear-mcp-spec-to-project.md`

**When to use:** After specs are approved (Step 2 complete), to create Linear issues.

**Owner:** AI (autonomous after sign-off)

**Key rules:**
- Read all spec files first (requirements.md, design.md, tasks.md)
- Query Linear for valid options
- Create 1:1 mapping between tasks.md and Linear issues
- Include proper estimates, labels, and traceability links

---

### Step 4: Developing Tasks (MANDATORY FOR ALL AGENTS)

See: `.kiro/standards/core/linear-mcp-task-development.md`

**When to use:** Every time you start working on a task

---

### Step 5: Feature Verification / Pre-PR Review

**When to use:** After all tasks are completed (Step 4), before PR review.

**Owners:** Product Manager (with Engineering support)

**Process:**
1. PM reviews completed feature against requirements.md
2. Engineering demonstrates implementation to PM
3. Verify all acceptance criteria are met
4. Identify any gaps or deviations from specs
5. PM and Engineering sign off on feature completeness
6. Proceed to PR review process

**Verification Checklist:**
- All tasks in tasks.md marked complete
- All Linear issues marked "Done"
- Feature meets requirements
- No critical bugs identified
- Ready for formal code review

**Quality Gate:** PM and Engineering must sign off before PR review begins

**Critical four-phase workflow (MANDATORY):**

1. **Phase 1 - Development:** Update to "In Progress" → Code → Dev Comment (summary of what was developed)
2. **Phase 2 - Testing:** Update to "Testing" → Test → First Results Comment → Fix/Retest (if needed) → Final Testing Comment
3. **Phase 3 - Commit:** Git Add → Git Commit (ONLY if tests pass)
4. **Phase 4 - Completion:** Update to "Done" → Final Comment (commit info + summary) → Update tasks.md

**Key points:**
- Four phases per task with version control mandatory
- Three status updates: "In Progress" → "Testing" → "Done"
- **Development comment AFTER coding** (documents what was implemented)
- **Testing phase allows fix/retest loops** with documented results
- **Git commit required ONLY after tests pass successfully**
- **NEVER commit code if tests fail**
- Comments required: Dev summary + First test results + Final testing results + Completion comment
- Commit ONLY happens after all tests pass
- "Done" ONLY after tests pass AND code is committed
- One task at a time
- No batching of status updates

---

## Agent Behavior Summary

- **Product Brief** (Step 0) defines high-level product scope (PM-owned, AI-assisted)
- **Linear Projects** (Step 1) represent features (derived from product brief)
- **Spec Creation** (Step 2) defines truth for each feature (PM + Engineering, AI-assisted)
- **Spec-to-Project** (Step 3) creates issues from approved specs (AI-owned)
- **Task Development** (Step 4) follows mandatory four-phase workflow (AI-owned)
- **Feature Verification** (Step 5) validates feature before PR review (PM + Engineering)
- GitHub hosts canonical documents
- Linear tracks execution
- **Development comment AFTER coding** (mandatory)
- **Testing allows fix/retest loops** with documented results
- Projects = feature intent (+ icon, sprint-sized, 1-2 weeks)
- Issues = 1:1 mapping with tasks.md
- Priorities: 2 (High), 3 (Medium), 4 (Low) — never 1 (Urgent)
- Labels classify work type
- Comments document each phase (dev, first test, final test, completion)

---

## Critical Development Flow

**Every task must follow this sequence:**

```
1. Find Linear issue for task
2. Update status to "In Progress"
3. Verify update succeeded
4. Write code
5. Add development comment (summary of what was implemented)
6. Update status to "Testing"
7. Verify update succeeded
8. Run tests
9. Add first test results comment
10. If tests FAIL: Fix code → Retest → Repeat until passing
11. Add final testing comment (what was fixed + final results)
12. Git add and commit changes
13. Verify commit succeeded
14. Update status to "Done"
15. Verify update succeeded
16. Add final comment (commit info + summary)
17. Update tasks.md checkbox
18. Move to next task
19. If tests persistently FAIL: Add failure comment, notify user, STOP (do NOT commit or mark Done)
```

**This flow is non-negotiable.**

---

## File Organization

```
.kiro/standards/core/
├── linear-mcp-rules.md                    # This file (overview)
├── linear-mcp-product-to-projects.md      # Product brief → Linear projects (FIRST STEP)
├── linear-mcp-spec-to-project.md          # Spec files → Linear issues
└── linear-mcp-task-development.md         # Task development workflow (MANDATORY)
```

---

## For More Details

- **Product to Projects:** Read `.kiro/standards/core/linear-mcp-product-to-projects.md`
- **Spec to Project:** Read `.kiro/standards/core/linear-mcp-spec-to-project.md`
- **Task Development:** Read `.kiro/standards/core/linear-mcp-task-development.md`

**All files contain normative rules that must be followed.**

Violating these rules constitutes incorrect agent behavior.
