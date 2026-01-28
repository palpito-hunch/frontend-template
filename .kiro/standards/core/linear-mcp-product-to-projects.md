# Linear MCP: Product Brief to Projects
---
inclusion: contextual
---

## Purpose

This document defines rules for breaking down a **Product Brief** into **Linear Projects**. This is the **first step** in the Linear MCP workflow, executed after a product brief is created.

**Key Concept**: A Product Brief → One or more Linear Projects (features). Each Linear Project = one feature in Kiro/GitHub/Claude.

For subsequent workflows:
- Creating issues within a project: See `linear-mcp-spec-to-project.md`
- Developing tasks: See `linear-mcp-task-development.md`

---

## AGENT QUICK START - PRODUCT TO PROJECTS

**When a Product Brief is ready, you MUST:**

1. **Read the Product Brief** to understand scope, goals, and deliverables
2. **Break down into sprint-sized projects** (1-2 weeks each)
3. **Present projects for review** before creating in Linear
4. **Query Linear for valid options:**
   - `list_teams()` - Get valid team names/IDs
   - `list_project_labels()` - See existing project labels
   - `list_projects()` - Check for existing projects
5. **Create projects** after PM approval
6. **Document manual follow-up tasks** (initiative linking, labels)

**NEVER:**
- Create projects without PM review and approval
- Use Urgent (1) priority for new projects
- Put emojis in project names (use icons instead)
- Skip the summary field
- Create projects larger than 2 weeks of work

---

## Core Principles

### Project Granularity

Projects should be **sprint-sized** (1-2 weeks of work):
- Single feature or cohesive functionality set
- Completable by 1-3 engineers
- Minimal external dependencies

### Naming & Organization

- **Names**: Clear, Title Case, 3-6 words (e.g., "User Registration Flow")
- **NO emojis in names** - use icons instead
- **Summaries**: Required, 1-2 sentences describing deliverables
- **Icons**: Use emoji shortcodes (`:lock:`, `:gear:`, `:bell:`)

---

## Project Creation Rules

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| `name` | Clear, descriptive, Title Case | "User Registration Flow" |
| `team` | Team name or ID | "Platform" |
| `description` | Comprehensive scope details | Full description of deliverables |
| `summary` | 1-2 sentence deliverable description | "Enables new users to create accounts via email or OAuth." |
| `state` | Default for new projects | "backlog" |
| `priority` | 2-4 only (High=2, Medium=3, Low=4) | 3 |
| `icon` | Shortcode format | ":lock:" |

### Priority Scale

**IMPORTANT**: High (2) is the maximum permitted priority for new projects.

| Priority | Value | Use Case |
|----------|-------|----------|
| ~~Urgent~~ | ~~1~~ | **NEVER USE** - Reserved for live incidents only |
| High | 2 | Blockers, critical path, foundational work |
| Medium | 3 | Important but non-blocking work |
| Low | 4 | Nice-to-have, analytics, future enhancements |

### Icon Selection Guide

Choose icons that represent the project domain:

| Category | Icons |
|----------|-------|
| Authentication & Security | `:lock:`, `:key:`, `:shield:`, `:unlock:`, `:closed_lock_with_key:` |
| User & Profile | `:bust_in_silhouette:`, `:pencil2:`, `:gear:`, `:bell:` |
| Wallet & Finance | `:moneybag:`, `:handshake:`, `:credit_card:`, `:bank:` |
| Compliance & Regulatory | `:clipboard:`, `:page_facing_up:`, `:warning:`, `:earth_americas:` |
| Analytics & Monitoring | `:bar_chart:`, `:label:`, `:mag:` |
| Account Management | `:control_knobs:`, `:wastebasket:` |
| Infrastructure | `:floppy_disk:`, `:link:`, `:ear:`, `:postbox:`, `:vertical_traffic_light:`, `:id:` |

---

## Linear MCP Functions Reference

### Create Project
```
Linear:create_project
- name: string (required)
- description: string (required)
- summary: string (required)
- team: string (required)
- state: "backlog" (default)
- priority: 2-4 (High/Medium/Low, never 1)
- icon: ":emoji:" shortcode
```

### Update Project
```
Linear:update_project
- id: string (required)
- name, description, summary, state, priority, icon (all optional)
```

### List Projects
```
Linear:list_projects
- team: filter by team
- state: filter by state
- query: search query
- limit: max 250
```

### List Project Labels
```
Linear:list_project_labels
- Returns all project labels with groups
```

---

## Workflow: Product Brief to Projects

### Step 1: Analyze the Product Brief

Read the product brief and identify:
- Core features/functionality areas
- Dependencies between features
- Technical components needed
- Sprint-sized work packages (1-2 weeks each)

### Step 2: Plan Projects

For each identified project:
- Define clear scope boundaries
- Ensure sprint-sized granularity
- Identify dependencies and ordering
- Map to appropriate initiative

### Step 3: Review Before Creating

**ALWAYS present projects for review before creating in Linear:**

```
Project Proposal:
- Name: [Title Case name]
- Icon: [shortcode]
- Priority: [2/3/4]
- Summary: [1-2 sentences]
- Key deliverables: [bulleted list]
- Initiative: [which initiative this maps to]
- Dependencies: [other projects this depends on]
- Labels: [proposed labels, including new ones if needed]
```

### Step 4: Create Projects (After Approval)

After PM approval:
1. Create each project with all required fields
2. Set priority (High=2 for blockers, Medium=3, Low=4)
3. Set status to Backlog
4. Assign to correct team

### Step 5: Document Manual Follow-Up

User must manually in Linear UI:
- Link projects to **one** domain initiative
- Assign project labels (including release labels like `mvp`)
- Create any new labels proposed

### Step 6: Verify

- List projects to confirm creation
- Check initiative assignments (done manually)
- Verify priorities are correct (no Urgent/1)

---

## Initiative Structure

Projects are linked to **one domain-specific initiative**. Release identification (MVP, v1.1, etc.) is handled via the `releases` label group.

### Initiative Assignment Rules

1. **Every project** links to exactly **one** domain initiative
2. Foundation/infrastructure projects link to infrastructure initiatives
3. Feature projects link to their domain initiative
4. **Release identification** is handled via labels (e.g., `mvp` label), NOT via initiatives

---

## Label Strategy

### Philosophy

Labels provide **cross-cutting segmentation** that complements (not duplicates) existing organization:
- **Initiatives** → Strategic grouping
- **Teams** → Ownership
- **Projects** → Feature grouping
- **Priority** → Importance (use Linear's built-in field)
- **Labels** → Cross-cutting concerns only

### Label Groups

Labels are organized into groups in Linear. **Do not use prefixes in label names**.

| Group | Purpose | Examples |
|-------|---------|----------|
| `type` | What kind of work | `library`, `infra`, `integration` |
| `scope` | Who uses it | `shared`, `internal` |
| `releases` | Release identification | `mvp` |
| `vendor` | External dependencies | `supabase`, `helius`, `privy` |

### When to Propose New Labels

Propose new labels if:
1. A **new vendor** is being integrated (add to `vendor` group)
2. A **new cross-cutting concern** emerges that spans multiple domains
3. The label provides filtering value not covered by initiatives/teams/projects

**Do NOT propose labels for**:
- Individual features or domains (use projects/initiatives)
- Priority or urgency (use Priority field)
- Team ownership (use Team field)
- One-off categorizations

### Label Proposal Format

```
Label Proposal:
- Name: [label name]
- Group: [type | scope | vendor | releases | ungrouped]
- Color: [hex code]
- Description: [what it's for]
- Rationale: [why existing labels don't cover this]
- Projects that would use it: [list]
```

---

## Project Labels vs Issue Labels

**Important**: Linear has two types of labels:
- **Project labels** - Applied to projects (managed here)
- **Issue labels** - Applied to issues within projects

The Linear MCP can:
- ✅ List project labels (`Linear:list_project_labels`)
- ✅ List issue labels (`Linear:list_issue_labels`)
- ✅ Create issue labels (`Linear:create_issue_label`)
- ❌ Create project labels (must be done manually in Linear UI)
- ❌ Delete labels (must be done manually in Linear UI)
- ❌ Assign labels to projects via MCP (must be done manually)

---

## Quick Reference Checklist

### Before Creating Projects
- [ ] Product brief analyzed and understood
- [ ] Projects are sprint-sized (1-2 weeks)
- [ ] Projects reviewed and approved by PM
- [ ] Names are Title Case, no emojis
- [ ] Summaries are 1-2 sentences
- [ ] Icons use shortcode format
- [ ] Priority is 2-4 (never 1/Urgent)
- [ ] Single initiative mapping identified
- [ ] Release label planned (e.g., `mvp`)
- [ ] New labels proposed if needed

### After Creating Projects
- [ ] All projects created successfully
- [ ] Priorities verified (no Urgent)
- [ ] Status set to Backlog
- [ ] Manual: Link to **one** domain initiative in Linear UI
- [ ] Manual: Assign release label (e.g., `mvp`) in Linear UI
- [ ] Manual: Assign other relevant labels in Linear UI
- [ ] Manual: Create any new labels in Linear UI

---

## Next Steps After Project Creation

Once projects are created in Linear:

1. **For each project**, create spec files in the repository:
   - `.kiro/specs/[feature]/requirements.md`
   - `.kiro/specs/[feature]/design.md`
   - `.kiro/specs/[feature]/tasks.md`

2. **Populate the project with issues** using the Spec-to-Project workflow:
   - See: `linear-mcp-spec-to-project.md`

3. **Develop tasks** using the Task Development workflow:
   - See: `linear-mcp-task-development.md`

---

## Anti-Patterns (Strictly Forbidden)

The agent MUST NOT:
- Create projects without PM review and approval
- Use Urgent (1) priority for new projects
- Put emojis in project names
- Skip the summary field
- Create mega-projects spanning multiple sprints
- Create projects that duplicate existing ones
- Skip querying Linear for valid teams/labels
- Create labels that duplicate initiatives or teams

---

## Summary (TL;DR)

- Product Brief → One or more Linear Projects
- Projects = sprint-sized (1-2 weeks)
- Always get PM approval before creating
- Priority: 2 (High), 3 (Medium), 4 (Low) — never 1
- Names: Title Case, no emojis
- Icons: Use shortcodes
- Manual follow-up: Initiative linking, labels
- Next: Create spec files, then populate with issues

Violating these rules constitutes incorrect agent behavior.
