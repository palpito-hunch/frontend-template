# Git Workflow Standards

## Purpose

This document defines the git workflow for the project. All code changes must follow this process to ensure code quality, maintain a clean history, and support staged deployments.

## Branching Strategy (Gitflow with Stages)

### Branch Hierarchy

```
main (prod)            # Production - deployed to production environment
├── uat                # UAT/Sandbox - deployed to staging for acceptance testing
│   └── develop        # Development - integration branch for features
│       ├── feature/*  # New features (e.g., feature/add-user-auth)
│       ├── fix/*      # Bug fixes (e.g., fix/balance-validation)
│       ├── refactor/* # Code refactoring (e.g., refactor/service-layer)
│       ├── docs/*     # Documentation changes (e.g., docs/api-reference)
│       └── chore/*    # Maintenance tasks (e.g., chore/update-dependencies)
└── hotfix/*           # Emergency fixes (branch from main, merge to all)
```

### Environment Stages

| Branch    | Environment | Purpose                          | Deploys To        |
| --------- | ----------- | -------------------------------- | ----------------- |
| `develop` | Development | Feature integration, dev testing | Dev server        |
| `uat`     | UAT/Sandbox | User acceptance testing, QA      | Staging server    |
| `main`    | Production  | Live production code             | Production server |

### Release Flow

```
feature/* → develop → uat → main
                ↓        ↓      ↓
              (dev)   (staging) (prod)
```

1. **Feature branches** → PR to `develop` (squash merge)
2. **develop** → PR to `uat` (merge commit, preserves history)
3. **uat** → PR to `main` (merge commit, after QA approval)

### Branch Naming Convention

- Use kebab-case: `feature/add-user-authentication`
- Be descriptive but concise: `fix/insufficient-balance-check`
- Include ticket/issue number if applicable: `feature/PROJ-123-market-resolution`

## Feature Development Workflow

### Creating a Feature Branch

```bash
# 1. Start from develop (NEVER from main or uat)
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# 2. Work on your feature, commit as needed
git add .
git commit -m "wip: initial implementation"
git commit -m "feat: complete core functionality"
git commit -m "fix: address edge case"

# 3. Keep branch up to date with develop
git fetch origin
git rebase origin/develop

# 4. Push and create PR to develop
git push -u origin feature/my-feature
```

### Pull Request to Develop

1. Create PR: `feature/my-feature` → `develop`
2. Fill out PR template
3. Wait for CI to pass
4. Get at least 1 approval
5. **Select "Squash and merge"** — creates single clean commit
6. Delete feature branch after merge

## Stage Promotion Workflow

### Promote develop → uat

When development testing is complete and features are ready for QA:

```bash
# Create PR: develop → uat
# Title: "Release: [version or date] to UAT"
```

**Requirements:**

- All tests passing
- QA approval to begin testing
- **Select "Create a merge commit"** (NOT squash) — preserves feature history

### Promote uat → main

After UAT sign-off and QA approval:

```bash
# Create PR: uat → main
# Title: "Release: [version or date] to Production"
```

**Requirements:**

- All tests passing
- Product owner approval
- QA sign-off
- **Select "Create a merge commit"** (NOT squash) — full audit trail

## Hotfix Workflow (Emergency Only)

For critical production issues that cannot wait for normal release cycle:

```bash
# 1. Branch from main (production)
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug-description

# 2. Fix the issue
git add .
git commit -m "fix: description of critical fix"

# 3. Create PR to main with "hotfix" label
git push -u origin hotfix/critical-bug-description
# Request expedited review

# 4. After merge to main, backport to uat and develop
git checkout uat
git pull origin uat
git merge origin/main
git push origin uat

git checkout develop
git pull origin develop
git merge origin/main
git push origin develop
```

## Pull Request Requirements

### Before Creating a PR

1. **All tests must pass locally**

   ```bash
   npm test
   npm run lint
   npm run type-check
   ```

2. **Branch is up to date**

   ```bash
   git fetch origin
   git rebase origin/develop  # For feature branches
   ```

3. **Commits are ready to be squashed**
   - WIP commits are fine — they'll be squashed
   - Final squash commit message should be meaningful

### PR Checklist

```
[ ] Tests pass (unit, integration, property-based)
[ ] No linting errors or warnings
[ ] TypeScript compiles without errors
[ ] Code follows project standards (see quick-reference.md)
[ ] Database migrations included if schema changed
[ ] Documentation updated if API changed
[ ] No secrets or credentials committed
[ ] PR targets correct branch (develop for features)
```

### PR Description Template

```markdown
## Summary

Brief description of what this PR does.

## Changes

- List of specific changes made

## Testing

How was this tested? What scenarios were covered?

## Related Issues

Closes #123 (if applicable)
```

### Review Requirements

- **Minimum 1 approval required** before merging
- Reviewer should verify:
  - Code follows standards
  - Tests adequately cover changes
  - No obvious security issues
  - No performance regressions

## Commit Message Convention

### Format

```
<type>: <short description>

[optional body]

[optional footer]
```

### Types

| Type       | Description                             |
| ---------- | --------------------------------------- |
| `feat`     | New feature                             |
| `fix`      | Bug fix                                 |
| `refactor` | Code refactoring (no functional change) |
| `test`     | Adding or updating tests                |
| `docs`     | Documentation changes                   |
| `chore`    | Maintenance, dependencies, configs      |
| `perf`     | Performance improvement                 |

### Examples

```
feat: add user authentication endpoint

fix: prevent negative balance in trade execution

refactor: extract calculations to separate service

test: add property tests for payout distribution

chore: update dependencies to latest versions
```

### Rules

- Use imperative mood: "add feature" not "added feature"
- Keep first line under 72 characters
- Reference issues in footer: `Closes #123`

## Protected Branch Rules

### All Protected Branches (main, uat, develop)

- **No direct commits** — all changes via PR
- **Required status checks:**
  - Tests must pass
  - Linting must pass
  - Type check must pass
- **Required reviews:** 1 approval minimum
- **No force pushes**

### Additional Rules by Branch

| Branch    | Who Can Merge  | Additional Requirements              |
| --------- | -------------- | ------------------------------------ |
| `develop` | Developers     | Tests pass, 1 approval               |
| `uat`     | Tech Lead / QA | Tests pass, QA approval              |
| `main`    | Tech Lead      | Tests pass, QA sign-off, PO approval |

## Merge Strategy Summary

| Merge Type        | Strategy         | Reason                          |
| ----------------- | ---------------- | ------------------------------- |
| feature → develop | **Squash**       | Clean single commit per feature |
| develop → uat     | **Merge commit** | Preserve feature history for QA |
| uat → main        | **Merge commit** | Full audit trail to production  |
| hotfix → main     | **Squash**       | Single fix commit               |

## Summary Table

| Action                            | Allowed | Not Allowed |
| --------------------------------- | ------- | ----------- |
| Direct commit to main/uat/develop |         | ❌          |
| PR without tests passing          |         | ❌          |
| PR without review                 |         | ❌          |
| Force push to protected branches  |         | ❌          |
| Feature branch from develop       | ✅      |             |
| Squash merge features to develop  | ✅      |             |
| Merge commit for stage promotions | ✅      |             |
| Hotfix branch from main           | ✅      |             |
| Rebase feature branch before PR   | ✅      |             |

---

**Remember:** The goal is maintainable, reviewable code with a clear path from development to production. Every change flows through the proper stages.
