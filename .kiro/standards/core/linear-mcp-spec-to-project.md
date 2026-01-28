# Linear MCP: Project and Issue Creation from Specs
---
inclusion: contextual
---

## Purpose

This document defines rules for creating Linear projects and issues from specification files. These rules apply when **initially setting up** a feature or project in Linear from spec files.

For rules about **developing tasks** (status updates during coding), see `linear_mcp_task_development.md`.

---

## AGENT QUICK START - PROJECT CREATION

**Before creating Linear artifacts from specs, you MUST:**

1. **Read all three spec files:**
   - `.kiro/specs/[feature]/requirements.md`
   - `.kiro/specs/[feature]/design.md`
   - `.kiro/specs/[feature]/tasks.md`

2. **Ask clarifying questions:**
   - Feature name, team assignment, emoji selection
   - Task-to-Issue mapping, estimates, priorities
   - Assignees, cycles, initial status

3. **Query Linear for valid options:**
   - `list_teams()` - Get valid team names/IDs
   - `list_issue_statuses()` - Get valid statuses for team
   - `list_issue_labels()` - See existing labels
   - `list_users()` - If assigning issues

4. **Only then proceed** with creating Linear artifacts

**NEVER:**
- Call Linear MCP without reading specs first
- Guess team names, statuses, or labels
- Skip clarifying questions
- Use local file paths (always use GitHub URLs)

---

## Canonical Sources of Truth

1. **requirements.md** — Defines *what must be true*
2. **design.md** — Defines *how correctness is achieved*
3. **tasks.md** — Defines *how work is executed*

Linear is **not** a source of truth.
Linear is a **projection layer** for execution tracking.

All references to these documents MUST point to their **canonical GitHub repository locations**, never to local file paths.

---

## Linear Object Mapping Rules

### 1. Project

**Rule P1 — One Project per Feature**
- The agent MUST create or use exactly one Linear Project per feature or initiative.
- Project name MUST match the feature name used in specs.

**Rule P2 — Project Description Content**
The Project description MUST contain:
- Goal (1–2 sentences)
- Non-negotiable invariants (bulleted)
- Success criteria (bulleted)
- Links to:
  - requirements.md (GitHub URL)
  - design.md (GitHub URL)

The Project description MUST NOT:
- Reproduce full requirements
- Reproduce design details
- Contain implementation steps

**Rule P3 — Project Icon / Emoji**
- Every new Project MUST include an icon or emoji.
- The emoji MUST be semantically related to the project scope (e.g., for notifications, for security, for infrastructure).
- Generic or unrelated emojis MUST NOT be used.

---

### 2. Issues

**Rule I1 — Issue Represents Direct Task Mapping**
- Each Issue MUST correspond directly to exactly one task from `tasks.md`.
- Issues MUST NOT be split into multiple Linear issues.
- Issues MUST NOT combine multiple tasks from `tasks.md`.
- The Issue represents a cohesive unit of work as defined in the canonical `tasks.md`.

**Rule I1a — Task Granularity Requirements**
- Tasks in `tasks.md` MUST be written at appropriate granularity to represent meaningful, user- or system-visible capabilities.
- If a task in `tasks.md` is too granular (e.g., "Write unit test for function X") or too large (>1 day), it MUST be revised in `tasks.md` first, not compensated for in Linear.
- Linear Issues reflect the granularity decisions made in the canonical `tasks.md`.
- The agent MUST NOT create Issues in Linear that don't exist as separate tasks in `tasks.md`.

**Rule I2 — Issue Naming**
Issue titles MUST:
- Be imperative and outcome-oriented
- Describe *what becomes true when done*
- Include the task number from tasks.md (e.g., "1. Create core notification card components")

Examples:
- "1. Implement single-notification state management"
- "2. Add keyboard-accessible dismiss behavior"

**Rule I3 — Issue Description Requirements**
Each Issue description MUST include:
- Short summary (1–3 sentences)
- Explicit non-goals (if relevant)

Issues MUST link to specs rather than restating them.

---

## Priority and Ordering Rules

**Rule O1 — Task-Driven Priority Assignment**
- The maximum automated priority level is **High**. **Urgent** priority MUST only be set manually by humans.
- Priority mapping: Tasks 1-5 = High, Tasks 6-10 = Medium, Tasks 11+ = Low.
- Main tasks should have high priority, property-test medium, and integration low

**Rule O2 — Issue Ordering**
- Issues SHOULD follow the same relative order as their corresponding tasks in `tasks.md`.

---

## Dependency and Linking Rules

**Rule L1 — Related Work Linking**
- If Issues are logically related, sequentially dependent, or mutually constraining, the agent MUST link them using Linear's issue-linking mechanisms.
- Linked relationships SHOULD reflect dependency direction where applicable.

---

## Traceability Rules

**Rule T1 — Forward Traceability**
Every Issue and sub-issue MUST trace to at least one Requirement.

**Rule T2 — Design Traceability**
Every Issue MUST reference the relevant Design section or correctness properties.

**Rule T3 — No Orphan Work**
If a task cannot be traced to a Requirement, it MUST NOT be created.

---

## Duplication and Drift Prevention

**Rule D1 — No Spec Duplication**
The agent MUST NOT:
- Copy full requirements into Linear
- Copy design documents into Linear
- Rephrase specs unless summarizing invariants

**Rule D2 — Spec Changes Flow Downward**
- If requirements or design change, the agent MUST update Issues.
- Issues MUST NOT override or reinterpret specs.

---

## Standards, Acronyms, and External References

**Rule G1 — Definitions for Acronyms and Standards**
- When introducing acronyms, global standards, or formal specifications (e.g., ISO, RFC, WCAG), the agent SHOULD include a link to an authoritative definition.
- Preferred sources include official standards bodies, specifications, or widely accepted references.

---

## Milestones and Labels for Testing Strategy

**Rule M1 — Mandatory Issue Classification**
To visually distinguish types of work, the agent MUST apply the following **labels** to every Issue:

- `implementation` — for implementation Issues
- `property-test` — for property-based test Issues
- `integration` — for integration-related Issues

An Issue MUST NOT exist without exactly one of these classifications.

---

## Estimation Rules

**Rule E1 — Mandatory Issue Estimates**
- Every Issue MUST include an estimate, expressed in hours or story points.

**Rule E2 — Estimation Guidelines**
The agent SHOULD apply the following defaults unless strong justification exists:

- Implementation Issues: **2–6 hours**
- Property-test Issues: **1–3 hours**
- Integration Issues: **2–6 hours**

Estimates are used for workload clarity and sprint planning and MUST be present before an Issue is marked as ready.

---

## Creation Ordering Rules

When creating Linear artifacts, the agent MUST follow this order:

1. Ensure Project exists and is correctly described (including emoji)
2. Create Issues mapped to tasks from `tasks.md`
3. Assign Issue priority based on `tasks.md` ordering
4. Create, label, estimate, and link Issues
5. Set initial Issue status (typically "Backlog" or "Todo")
6. Update Project status to "Planned"

The agent MUST create Issues in the proper order as defined above.

---

## Project Status Management Rules

**Rule S1 — Project Status Update After Issue Creation**
- Once all Issues from the feature specs have been created and linked to the Project, the agent MUST update the Project status to "Planned".
- This status change MUST occur BEFORE providing the project update to the user.
- The "Planned" status indicates that all specification work has been loaded into Linear and the project is ready for development.

---

## Project Update Rules

**Rule U1 — Initial Project Update**
- Once all Projects and Issues are created and linked, the agent MUST provide a Project update over the chat.

**Rule U2 — Project Update Content**
The update MUST include:
- High-level project status
- Summary of scope loaded into Linear
- Confirmation of traceability to requirements and design
- Any notable risks, dependencies, or open questions

---

## Anti-Patterns (Strictly Forbidden)

The agent MUST NOT:
- Create "mega issues" covering the entire project
- Use Issues as documentation dumps
- Track progress in Project descriptions
- Encode design decisions inside task titles
- Create Issues for work not specified in `tasks.md`

---

## Completion Rules

**Rule C1 — Issue Completion**
An Issue MAY be closed only when:
- All work for the Issue is completed
- Acceptance criteria implied by Requirements are met

**Rule C2 — Project Completion**
A Project MAY be closed only when:
- All Issues are closed
- All Requirements are satisfied

---

## Summary (TL;DR)

- Specs define truth
- GitHub hosts canonical documents
- Linear tracks execution
- Projects = intent (+ emoji)
- Issues = 1:1 mapping with tasks.md
- Priorities follow tasks.md ordering
- Labels classify work type
- Estimates enable planning
- Links express dependencies
- Traceability to requirements is mandatory
- Project status set to "Planned" after all issues created

Violating these rules constitutes incorrect agent behavior.
