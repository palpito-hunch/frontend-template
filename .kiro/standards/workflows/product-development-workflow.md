# Product Development Workflow

**ADR**: [ADR-0005](.kiro/memory/decisions/0005-ways-of-working-product-development.md)

## Overview

This workflow defines the six-phase product development process from problem identification through release tracking.

**Key Concept**: Problem → Product Brief → Linear Projects → Specs → Issues → Development → Verification → Release

One product brief may result in multiple projects. Each Linear Project = one feature in Kiro/GitHub/Claude.

---

## Phase 0: Product Brief Creation (FOUNDATION)

**Owner:** Product Manager

**When to use:** When a product manager identifies a problem, requirement, or opportunity to address.

### Process

1. Product manager identifies a problem, requirement, or opportunity
2. PM drafts initial product brief describing the functionality
3. PM collaborates with AI to refine and improve the brief
4. PM curates extensively to ensure clarity and completeness
5. Final brief is reviewed and approved before proceeding

### Product Brief Requirements

- Clear problem statement or opportunity description
- Well-defined scope and objectives
- User stories or use cases
- Success criteria and acceptance criteria
- Constraints and dependencies
- Out-of-scope items explicitly stated

### Key Principles

- **AI-assisted, PM-curated**: AI helps draft and refine, but PM owns the final document
- **Depth over speed**: Take time to create a solid, well-defined brief
- **Clarity is paramount**: The brief must explain clearly what needs to be created
- **No ambiguity**: Vague requirements lead to rework; resolve unclear items before proceeding

### Quality Gate

The product brief must be complete and approved before moving to Phase 1 (breaking down into projects).

---

## Phase 1: Product Brief → Projects

**Owner:** PM + AI

**When to use:** After a product brief is created, as the first step before any development work.

**Rules file:** `.kiro/standards/core/linear-mcp-product-to-projects.md`

### Process

1. Analyze the product brief to identify features
2. Break down into sprint-sized projects (1-2 weeks each)
3. Present projects for PM review and approval
4. Query Linear for valid teams and labels
5. Create projects after approval
6. Document manual follow-up (initiative linking, labels)

### Key MCP Tools

- `list_teams()` - Get valid team names/IDs
- `list_projects()` - Check for existing projects
- `list_project_labels()` - See existing project labels
- `create_project()` - Create new projects

### Project Creation Rules

- **Names**: Title Case, no emojis (use icons instead)
- **Priority**: 2 (High), 3 (Medium), 4 (Low) — never 1 (Urgent)
- **Summaries**: Required, 1-2 sentences
- **Icons**: Use shortcode format (`:lock:`, `:gear:`)
- **State**: Default to "backlog"
- **Scope**: Sprint-sized (1-2 weeks of work)

---

## Phase 2: Spec Creation / Feature Refinement

**Owner:** Product Manager + Engineering (AI-assisted)

**When to use:** After a project exists in Linear (Phase 1 complete), before creating issues.

This is a collaborative refinement session—similar to a "super-charged Scrum refinement"—where PM and Engineering work together with AI assistance to define the complete specifications.

### Process

1. Review the Linear project definition (name, description, summary)
2. PM and Engineering collaborate to define requirements
3. Use AI to draft and refine specification documents
4. Create `requirements.md` - what must be true (acceptance criteria)
5. Create `design.md` - how correctness is achieved (technical approach)
6. Create `tasks.md` - how work is executed (task breakdown)
7. Review and iterate until specs are complete and unambiguous
8. PM and Engineering sign off on specifications

### Spec File Locations

```
.kiro/specs/[feature]/
├── requirements.md    # What must be true
├── design.md          # How correctness is achieved
└── tasks.md           # How work is executed
```

**Linear Project Linkage:** The `requirements.md` file MUST include a reference to the associated Linear project at the top of the document. This establishes traceability between specs and project tracking.

Example:
```markdown
# Feature Name Requirements

**Linear Project:** [Project Name](https://linear.app/team/project/PROJECT-ID)

## Requirements
...
```

### Key Principles

- **Collaborative**: PM brings product context, Engineering brings technical feasibility
- **AI-assisted**: AI helps draft, organize, and identify gaps—humans curate and approve
- **Thorough refinement**: Resolve ambiguities before development starts
- **No rushing**: Quality specs prevent rework during development

### Deep Refinement with AskUserQuestion

AI agents MUST use the `AskUserQuestion` tool extensively during this phase to:
- Probe for edge cases that may not be immediately obvious
- Clarify ambiguous requirements before they become implementation problems
- Explore alternative approaches and trade-offs
- Validate assumptions about user behavior and system constraints
- Identify potential failure modes and error scenarios
- Ensure completeness of acceptance criteria

This deep questioning process is essential for producing well-defined specs that minimize rework during development. The goal is to surface and resolve issues during refinement, not during coding.

### Quality Gate

Both PM and Engineering must sign off on specs before proceeding to Phase 3.

---

## Phase 3: Spec-to-Project (Issues from Specs)

**Owner:** AI (autonomous after sign-off)

**When to use:** After specs are approved (Phase 2 complete), to create Linear issues.

**Rules file:** `.kiro/standards/core/linear-mcp-spec-to-project.md`

### Process

1. Read all spec files (requirements.md, design.md, tasks.md)
2. Query Linear for valid options (teams, statuses, labels)
3. Update Linear Project with appropriate metadata
4. Create Issues with 1:1 mapping to tasks.md entries
5. Set priorities, estimates, and labels
6. Update Project status to "Planned"

### Key MCP Tools

- `list_teams()` - Get valid team names/IDs
- `list_issue_statuses()` - Get valid statuses for team
- `list_issue_labels()` - See existing labels
- `list_users()` - If assigning issues
- `update_project()` - Update project metadata
- `create_issue()` - Create issues from tasks

---

## Phase 4: Task Development (Mandatory)

**Owner:** AI

**When to use:** Every time an agent starts working on a task.

**Rules file:** `.kiro/standards/core/linear-mcp-task-development.md`

### Mandatory Four-Phase Workflow

| Phase | Action | Linear Status | Comments |
|-------|--------|---------------|----------|
| 1. Development | Update status → Code → Comment | "In Progress" | Summary of what was developed |
| 2. Testing | Update status → Test → Comment → Fix/retest loop → Final comment | "Testing" | First test results + Final results (with fixes if any) |
| 3. Commit | Git add → Git commit (only if tests pass) | (no change) | None |
| 4. Completion | Update status → Final comment → Update tasks.md | "Done" | Commit info + final summary |

### Key MCP Tools

- `list_issues()` - Find the correct issue for the task
- `update_issue()` - Update status at each phase boundary
- `create_comment()` - Document development, testing iterations, and completion

### Critical Rules

- Development comment comes AFTER coding is complete
- Testing phase allows iterative fix/retest cycles with documented results
- Commits ONLY after tests pass
- "Done" ONLY after tests pass AND code is committed
- One task at a time (no batching)

---

## Phase 5: Feature Verification / Pre-PR Review

**Owner:** Product Manager (with Engineering support)

**When to use:** After all tasks are completed (Phase 4), before creating/submitting PRs for review.

### Process

1. PM reviews the completed feature against requirements.md
2. Engineering demonstrates the implementation to PM
3. Verify all acceptance criteria are met
4. Verify the feature works as specified in the product brief
5. Identify any gaps, issues, or deviations from specs
6. Document any necessary follow-up items or known issues
7. PM and Engineering sign off on feature completeness
8. Proceed to PR review process

### Verification Checklist

- [ ] All tasks in tasks.md are marked complete
- [ ] All Linear issues are marked "Done"
- [ ] Feature meets requirements defined in requirements.md
- [ ] Implementation aligns with design.md
- [ ] No critical bugs or regressions identified
- [ ] Feature is ready for formal code review

### Key Principles

- **PM-led**: Product Manager drives verification against original requirements
- **Engineering support**: Engineers demonstrate and explain implementation details
- **Quality gate**: Feature must be verified before PR review begins
- **Early feedback**: Catch issues before formal review process

### Quality Gate

PM and Engineering must sign off that the feature is complete and ready for PR review.

---

## Phase 6: Release Tracking / Environment Promotion (TODO)

**Owner:** Engineering (with PM visibility)

**When to use:** After feature verification (Phase 5), through production deployment.

### Promotion Flow

```
Feature Branch → Develop → UAT → Production
     ↓              ↓        ↓        ↓
  PR Created    PR Merged  PR Merged  PR Merged
     ↓              ↓        ↓        ↓
 Linear: "Final Review" → "In Develop" → "In UAT" → "Released"
```

### Process

1. **Feature → Develop**: Create PR from feature branch to develop
   - Linear Project status: "Final Review"
   - PR review and approval
   - Squash merge to develop
   - Linear Project status: "In Develop"

2. **Develop → UAT**: Promote to UAT branch
   - Create PR from develop to UAT
   - Linear Project status: "In UAT"
   - QA/acceptance testing in staging environment
   - PM sign-off on UAT

3. **UAT → Production**: Promote to main/production
   - Create PR from UAT to main
   - Final approval
   - Merge to production
   - Linear Project status: "Released"
   - Tag release version

### Linear Project Status Progression

| Stage | Linear Status | Environment | Gate |
|-------|---------------|-------------|------|
| After Phase 5 | "Final Review" | - | Feature verified |
| Merged to develop | "In Develop" | Dev server | PR approved |
| Merged to UAT | "In UAT" | Staging server | QA testing |
| Merged to main | "Released" | Production | Final approval |

### TODO - Implementation Required

- [ ] Create automation to update Linear status on PR merge events
- [ ] Define rollback procedures and Linear status handling
- [ ] Document hotfix flow and Linear tracking
- [ ] Integrate with CI/CD pipeline notifications

---

## Source of Truth Model

| Aspect | Authoritative Source |
|--------|---------------------|
| Requirements (what must be true) | `requirements.md` |
| Design (how correctness is achieved) | `design.md` |
| Work breakdown (tasks to execute) | `tasks.md` |
| Execution status | Linear |
| Time tracking / estimates | Linear |
| Development comments / discussions | Linear |
| Cycles / sprints | Linear |

**Principle:** Spec files define WHAT needs to be done. Linear tracks HOW execution is progressing.

---

## Constraints

1. **Linear team setup required** - Valid Linear team, statuses, and labels must exist before agents can interact with Linear.

2. **Single agent at a time** - Only one agent should work on a task at a time to avoid status update conflicts.

3. **Issues require spec files** - Linear projects can be created without spec files, but Linear issues within projects should only be created once spec files (requirements.md, design.md, tasks.md) are ready.

---

## Enforcement

These workflows are enforced through normative rule files in `.kiro/standards/core/`:

```
.kiro/standards/core/
├── linear-mcp-rules.md                 # Overview and quick reference
├── linear-mcp-product-to-projects.md   # Product brief → projects (FIRST STEP)
├── linear-mcp-spec-to-project.md       # Spec files → issues
└── linear-mcp-task-development.md      # Task development workflow rules
```

Violating these rules constitutes incorrect agent behavior. Agents must follow the detailed rules in these files exactly.
