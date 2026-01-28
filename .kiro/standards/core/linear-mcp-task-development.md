# Linear MCP: Task Development Workflow
---
inclusion: always
---

## Purpose

This document defines **mandatory workflow rules** for agents when developing tasks. These rules ensure real-time visibility of work-in-progress by requiring status updates in Linear at each phase: development, version control commit, testing, and completion.

For rules about **creating projects and issues from specs**, see `linear-mcp-spec-to-project.md`.

---

## CRITICAL - MANDATORY WORKFLOW

**Every time you develop a task, you MUST follow this four-phase workflow:**

### **Phase 1: Development (MANDATORY)**

1. **Update Linear status to "In Progress"** before writing any code
   - Call `update_issue()` to set status to "In Progress"
   - Confirm the update succeeded
   - Write code to complete the task
   - **After coding is complete**, add a comment with **development summary** using `create_comment()`
   - The comment MUST include: what was implemented, files modified, key decisions

### **Phase 2: Testing (MANDATORY)**

2. **Update Linear status to "Testing"** after development comment is added
   - Call `update_issue()` to set status to "Testing"
   - Confirm the update succeeded
   - Run tests
   - Add a comment with **first test results** using `create_comment()`
   - **If tests fail**: Fix the code, run tests again (repeat as needed)
   - Add a **final testing comment** with: what was fixed (if anything) + final test results

### **Phase 3: Commit (MANDATORY)**

3. **Commit code changes** after all tests pass successfully
   - Stage all relevant code changes
   - Write a clear, descriptive commit message following conventional commits format
   - Commit the changes to the repository
   - Confirm the commit succeeded
   - ONLY commit if ALL tests passed

### **Phase 4: Completion (MANDATORY)**

4. **Update Linear status to "Done"** after committing code
   - Call `update_issue()` to set status to "Done"
   - Confirm the update succeeded
   - Add a **final comment** with: commit info (hash, message) + final summary of the work
   - Update tasks.md checkbox from `[ ]` to `[x]`
   - ONLY THEN move to next task

**NEVER:**
- Start coding without updating Linear status first
- Skip the development comment after coding
- Run tests without moving to "Testing" first
- Skip the first test results comment
- Skip the final testing comment
- Commit code before all tests pass successfully
- Mark as "Done" without committing code first
- Skip the final completion comment
- Commit code when tests fail

---

## Status Management Rules (MANDATORY)

**Rule S1 — Pre-Development Status Update (MANDATORY)**
- **BEFORE** writing ANY code for a task, the agent MUST update the corresponding Linear Issue status to "In Progress".
- The agent MUST call `update_issue()` with the appropriate status parameter.
- The agent MUST wait for confirmation that the status update succeeded.
- Code development MUST NOT begin until the Linear status has been successfully updated.
- This ensures real-time visibility of work-in-progress across the team.

**Rule S2 — Development Comment (MANDATORY)**
- **AFTER** completing all code development (while still in "In Progress"), the agent MUST add a comment using `create_comment()`.
- The comment MUST include:
  - Summary of what was implemented
  - Key technical details or decisions
  - Files modified or created
  - Any important notes for reviewers
- This comment documents the development work before moving to testing.

**Rule S3 — Pre-Testing Status Update (MANDATORY)**
- **AFTER** adding the development comment, the agent MUST update the corresponding Linear Issue status to "Testing".
- The agent MUST call `update_issue()` to set status to "Testing".
- The agent MUST wait for confirmation that the status update succeeded.
- Testing MUST NOT begin until the Linear status has been successfully updated to "Testing".
- This signals that code is ready for testing.

**Rule S4 — Run Tests and First Results Comment (MANDATORY)**
- After moving to "Testing", the agent MUST run all relevant tests.
- The agent MUST add a comment with **first test results** using `create_comment()`.
- The comment MUST include:
  - Test execution summary (passed/failed)
  - Which tests were run
  - Any errors or failures encountered

**Rule S5 — Fix/Retest Loop (IF NEEDED)**
- If tests fail, the agent MUST fix the code and run tests again.
- This fix/retest cycle can repeat as many times as needed.
- The agent remains in "Testing" status during this loop.
- Each iteration should focus on fixing the identified issues.

**Rule S6 — Final Testing Comment (MANDATORY)**
- After all tests pass, the agent MUST add a **final testing comment** using `create_comment()`.
- The comment MUST include:
  - What was fixed (if anything) during the fix/retest loop
  - Final test results (all passed)
  - Test output summary
- If no fixes were needed, state "No fixes required - all tests passed on first run."

**Rule S7 — Version Control Commit (MANDATORY)**
- **IMMEDIATELY AFTER** all tests pass, the agent MUST commit the changes to version control.
- The agent MUST:
  - Stage all relevant code changes using `git add`
  - Write a clear, descriptive commit message following conventional commits format (feat/fix/docs/chore/refactor/test/perf/ci)
  - Include the task number or Linear issue reference in the commit message
  - Commit the changes to the repository
  - Verify the commit succeeded
- The commit message MUST be meaningful and describe what was changed and why.
- Code MUST ONLY be committed if ALL tests passed successfully.

**Rule S8 — Post-Commit Status Update (MANDATORY)**
- **IMMEDIATELY AFTER** committing code changes, the agent MUST update the corresponding Linear Issue status to "Done".
- The agent MUST call `update_issue()` to set status to "Done".
- The agent MUST wait for confirmation that the status update succeeded.
- The agent MUST NOT move to the next task until the status has been successfully updated to "Done".

**Rule S9 — Final Completion Comment (MANDATORY)**
- After moving to "Done", the agent MUST add a **final comment** using `create_comment()`.
- The comment MUST include:
  - Commit information (hash, commit message)
  - Final summary of the completed work
- This comment serves as the definitive record of task completion.

**Rule S10 — Status Update Timing**
- Status updates MUST occur at the correct phase boundaries:
  - "In Progress" → Before coding
  - "Testing" → After development comment, before testing
  - "Done" → After commit succeeds
- Status updates MUST NOT be batched or deferred.
- If multiple tasks are being worked on sequentially, each task's status MUST be updated individually at each phase.

**Rule S11 — Status Update Verification**
- After calling `update_issue()`, the agent SHOULD verify the status change by checking the response.
- If the status update fails, the agent MUST retry or notify the user before proceeding.

**Rule S12 — Persistent Test Failures**
- If tests continue to fail after multiple fix attempts and the agent cannot resolve the issue:
  - The issue MUST remain in "Testing" status
  - The agent MUST add a comment summarizing all attempted fixes and remaining failures
  - The agent MUST notify the user about the persistent test failures
  - The agent MUST NOT commit code when tests fail
  - The agent MUST NOT move to "Done" status
  - The agent MUST NOT proceed to the next task

---

## Development Workflow Ordering (MANDATORY)

When working on a task, the agent MUST follow this strict sequence:

### **Phase 1: Development**
1. **Find Linear issue** → Use `list_issues()` to find the task
2. **Update status to "In Progress"** → Call `update_issue()`
3. **Verify update** → Confirm the status change succeeded
4. **Write code** → Complete all code changes for the task
5. **Add development comment** → Call `create_comment()` with implementation summary

### **Phase 2: Testing**
6. **Update status to "Testing"** → Call `update_issue()`
7. **Verify update** → Confirm the status change succeeded
8. **Run tests** → Execute relevant tests
9. **Add first test results comment** → Call `create_comment()` with test results
10. **If tests FAIL** → Fix code → Run tests again → Repeat until passing
11. **Add final testing comment** → Call `create_comment()` with fixes (if any) + final results

### **Phase 3: Commit**
12. **Stage changes** → Use `git add` to stage all relevant files (only after tests pass)
13. **Commit changes** → Create commit with descriptive message following conventional commits format
14. **Verify commit** → Confirm the commit succeeded

### **Phase 4: Completion**
15. **Update status to "Done"** → Call `update_issue()`
16. **Verify update** → Confirm the status change succeeded
17. **Add final comment** → Call `create_comment()` with commit info + final summary
18. **Update tasks.md** → Change checkbox from `[ ]` to `[x]`
19. **Move to next task** → Only proceed after all steps complete

**This sequence MUST NOT be reversed or parallelized.**

---

## Finding the Correct Linear Issue

**Rule F1 — Issue Discovery**
- Before starting development, the agent MUST find the correct Linear issue for the task.
- Use `list_issues()` with a search query based on the task number or description.
- Verify the issue matches the task from `tasks.md`.

**Rule F2 — Task-Issue Correspondence**
- Ensure the Linear issue corresponds to the task you're working on.
- Verify by checking the issue title contains the task number (e.g., "4.2" for task 4.2).
- If no issue exists, notify the user - do NOT create issues during development.

---

## Status Update Examples

### Example 1: Starting a task and completing development

```typescript
// CORRECT: Update status before coding
const issue = await list_issues({ query: "4.2 keyboard navigation" });
await update_issue({ id: issue.id, state: "In Progress" });
// Status updated - NOW begin coding

// ... write all code for the task ...

// AFTER coding is complete, add development comment (still in "In Progress")
await create_comment({
  issueId: issue.id,
  body: `## Development Complete

### Implementation Summary
- Implemented keyboard navigation test for dismiss button
- Added property-based test using fast-check
- Test validates Requirements 6.1, 6.2, 8.5

### Technical Details
- Tests both Enter and Space key activation
- Validates focus indicators (focus:ring-2, focus:ring-blue-500)
- Verifies DOM focus state and button activation

### Files Modified
- \`src/components/__tests__/notification-card.property.test.tsx\` (lines 311-410)`
});
```

### Example 2: Move to testing and run tests

```typescript
// CORRECT: Move to "Testing" and run tests
await update_issue({ id: issue.id, state: "Testing" });

// Run tests
const testResult = await runTests();

// Add first test results comment
await create_comment({
  issueId: issue.id,
  body: `## First Test Results

**Status**: All tests passed

### Tests Executed
- Property 14: Keyboard Navigation
- Duration: 10.2s
- Runs: 20 property test iterations

### Test Output
\`\`\`
Property 14: Keyboard Navigation  10257ms
\`\`\``
});

// Since all tests passed on first run, add final testing comment
await create_comment({
  issueId: issue.id,
  body: `## Final Testing Results

**No fixes required** - all tests passed on first run.

### Final Results
- All 20 property test iterations passed
- Requirements 6.1, 6.2, 8.5 validated`
});
```

### Example 3: Tests fail, fix, and retest

```typescript
// First test run - tests fail
await create_comment({
  issueId: issue.id,
  body: `## First Test Results

**Status**: Tests failed

### Failed Test
- Property 14: Keyboard Navigation
- **Error**: Expected button to be focusable, but tabIndex was -1`
});

// Fix the code
// ... make fixes ...

// Run tests again - now they pass

// Add final testing comment with fixes
await create_comment({
  issueId: issue.id,
  body: `## Final Testing Results

### Fixes Applied
- Added \`tabIndex={0}\` to dismiss button component
- Updated button accessibility attributes

### Final Results
- All tests now passing
- Property 14: Keyboard Navigation - PASSED`
});
```

### Example 4: Commit and complete

```typescript
// CORRECT: Tests passed, commit code, update status to "Done", and add final comment
await bash({
  command: 'git add src/components/__tests__/notification-card.property.test.tsx && git commit -m "test(notification-card): add keyboard navigation property test\n\nImplement property-based test for dismiss button keyboard navigation.\nValidates Requirements 6.1, 6.2, 8.5.\n\nCo-Authored-By: Claude Opus 4.5 <noreply@anthropic.com>"'
});

// Update status to "Done"
await update_issue({ id: issue.id, state: "Done" });

// Add final completion comment
await create_comment({
  issueId: issue.id,
  body: `## Task Complete

### Commit Information
- **Hash**: a1b2c3d
- **Message**: test(notification-card): add keyboard navigation property test

### Final Summary
Implemented property-based test for keyboard navigation on the dismiss button. Test validates Enter and Space key activation, focus indicators, and DOM focus state. All tests passing.`
});

// Update tasks.md checkbox
// NOW move to next task
```

---

## Query Linear for Valid Options

Before updating status, agents SHOULD verify valid status names:

```typescript
// Get valid statuses for the team
const statuses = await list_issue_statuses({ team: "playground" });
// statuses will include: "Backlog", "Todo", "In Progress", "Testing", "Done", etc.
```

---

## Anti-Patterns (Strictly Forbidden)

The agent MUST NOT:
- Start writing code before updating Linear status to "In Progress"
- Skip the development comment after coding is complete
- Move to "Testing" without first adding the development comment
- Run tests before updating Linear status to "Testing"
- Skip the first test results comment
- Skip the final testing comment
- Commit code before all tests pass successfully
- Skip the version control commit step after tests pass
- Mark a task as "Done" before committing code
- Mark a task as "Done" before tests pass
- Skip the final completion comment when moving to "Done"
- Mark as "Done" when tests fail (stay in "Testing" and fix)
- Commit code when tests fail
- Batch status updates (each phase must be updated individually)
- Update status without verifying it succeeded
- Commit without a clear, descriptive commit message

---

## Critical Development Flow (TL;DR)

**Mandatory four-phase sequence for every task:**

```
Phase 1: In Progress → Code → Dev Comment
Phase 2: Testing → Test → First Results Comment → Fix/Retest (if needed) → Final Testing Comment
Phase 3: Commit → Git Add → Git Commit (ONLY if tests pass)
Phase 4: Done → Final Comment (commit info + summary) → Update tasks.md → Next Task
```

**Complete Flow:**
```
1. Find Issue
2. Update to "In Progress" → Verify
3. Code
4. Add Development Comment (what was implemented)
5. Update to "Testing" → Verify
6. Run Tests
7. Add First Test Results Comment
8. If tests FAIL: Fix code → Retest → Repeat until passing
9. Add Final Testing Comment (what was fixed + final results)
10. Git Add → Git Commit → Verify
11. Update to "Done" → Verify
12. Add Final Comment (commit info + summary)
13. Update tasks.md → Next Task
14. If ALL tasks complete: Update Project to "Final Review" → Verify → Notify User
```

**Key Points:**
- Four phases per task: "In Progress" → "Testing" → Commit → "Done"
- **Development comment AFTER coding** (still in "In Progress")
- **Testing phase allows fix/retest loops** with documented results
- **First test results comment + Final testing comment** required
- **Version control commit MANDATORY** after tests pass
- **NEVER commit code if tests fail**
- Three status updates per task: "In Progress" → "Testing" → "Done"
- Final completion comment includes commit info + summary
- Commit ONLY happens after all tests pass
- "Done" ONLY after tests pass AND code is committed
- One task at a time
- No batching
- **Project status to "Final Review" when all tasks complete**

**Violating these rules constitutes incorrect agent behavior.**

---

## Integration with tasks.md

**Rule T1 — Update tasks.md Checkbox**
- After marking an issue as "Done" in Linear, the agent MUST update the corresponding checkbox in `.kiro/specs/[feature]/tasks.md`.
- Change `- [ ]` to `- [x]` for the completed task.
- This keeps the spec file in sync with Linear status.
- This MUST only happen after tests pass and status is "Done".

---

## Project Status Management (MANDATORY)

**Rule P1 — Project Status Update After All Tasks Complete**
- When ALL required tasks from `tasks.md` have been completed (all checkboxes marked `[x]` and all issues marked "Done"), the agent MUST update the Project status to "Final Review".
- The agent MUST call `update_project()` to set the project status to "Final Review".
- The agent MUST verify the project status update succeeded.
- This status change indicates that all development and testing work is complete and the project is ready for final review.
- This MUST happen AFTER the last task's status is updated to "Done" and its checkbox is marked in tasks.md.

**Rule P2 — Verification of Completion**
- Before updating the project status to "Final Review", the agent MUST verify:
  - All tasks in tasks.md are checked `[x]`
  - All corresponding Linear issues are marked "Done"
  - All tests have passed successfully
- If any task is incomplete, the project status MUST NOT be updated.

**Rule P3 — Project Completion Notification**
- After updating the project status to "Final Review", the agent MUST notify the user that:
  - All tasks have been completed
  - All tests have passed
  - The project is ready for final review
  - The project status has been updated in Linear

---

## Summary

This four-phase workflow ensures:
- Real-time visibility of work-in-progress at each phase
- **Development is documented before testing begins**
- **Test results are documented at first run AND after fixes**
- **Iterative fix/retest cycles are supported** within the Testing phase
- **All code changes committed to version control ONLY after tests pass**
- **Failed tests prevent code commits** - only working, tested code is committed
- Clear audit trail: development comment → first test results → fixes → final results → commit → completion
- Proper synchronization between Linear and spec files
- Project status updated to "Final Review" when all tasks complete

**The workflow is non-negotiable.** Every agent developing tasks must follow these rules exactly.
