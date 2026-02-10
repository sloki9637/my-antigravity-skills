---
description: Feature Implementation Workflow
---

# Feature Implementation Workflow

This workflow is designed to create **natural pull** from planning → issue → code → review → history.  
Skipping steps should feel harder than following them.

---

## 1. Plan First (Mandatory)

- Define the implementation approach
- Break work into phases
- Identify risks and dependencies
- No code is written before a plan exists.
- The plan is the **single source of truth** until the issue is created.

---

## 2. Issue Registration (Gravity Anchor)

- Immediately after plan approval, call the `git-issue-formatter` skill.
- Register an issue on **GitHub or GitLab**.

### Rules
- Issue content **must be written in Korean**
- One plan → one issue
- The issue becomes the **central gravity point** for:
  - Branch naming
  - Commits
  - Pull Requests
  - Review discussion

### Required Issue Sections
- Background / Context
- Goal
- Plan summary (from planner)
- Acceptance Criteria
- Risks & Dependencies
- Test Plan (draft, TODO allowed)

---

## 3. Development (Orbit Around Issue)

- Create a branch linked to the issue number:

```

feat/#123-short-description
fix/#456-bug-description

```

- All development work must be done on this branch.
- All changes must stay within the scope defined by the issue.
- If scope or plan changes:
  - Update the issue first
  - Then continue development

---

## 4. Self Review Before Commit (Mandatory)

- Before committing, verify the implementation against:
  - Issue Goal
  - Acceptance Criteria
- Any deviation must be:
  - Fixed, or
  - Explicitly documented in the issue or PR

---

## 5. Code Review (Stabilization)

- After implementation, run the `code-reviewer` skill.
- Address findings in the following order:
  1. **CRITICAL**
  2. **HIGH**
  3. MEDIUM (when feasible)

- Code is not considered ready if CRITICAL or HIGH issues remain unresolved.
- Review findings should be summarized for PR review usage.

---

## 6. Commit & Push (Historical Record)

- Use the `git-commit-formatter` skill for all commits.
- Commit messages:
  - Must include the issue number
  - Must be written in **Korean**
  - Must follow the conventional commit format

### Example
```

feat: 초기 줌아웃을 위한 InteractiveViewer 컨트롤러 추가 (#123)

```

- When pushing a new branch, always use:
```

git push -u origin <branch-name>

```

---

## 7. Pull Request Creation (Execution Log)

- Use the `git-pr-formatter` skill for all prs.
- Create a Pull Request linked to the issue.
- PR description must include:
  - Summary of changes
  - Acceptance Criteria checklist
  - Test results or TODOs
  - Link to the issue

---

## 8. PR Review: Code Review Summary (Mandatory)

- In the PR review, add a **Code Review Summary** comment.
- This comment represents the final review record.

### Review Summary Must Include
- Review scope
- CRITICAL / HIGH issues found (or an explicit statement that none exist)
- Key design decisions
- Trade-offs or intentional decisions

### Rules
- Review content **must be written in Korean**
- All unresolved concerns must be:
  - Addressed, or
  - Explicitly justified in the review

---

## 9. Merge & Closure

- The issue can be closed only when:
  - PR is merged
  - Acceptance Criteria are fully satisfied
  - Code Review Summary is present in the PR

---

## Core Principles

- **Plan pulls the Issue**
- **Issue pulls the Code**
- **Code pulls the Review**
- **Review pulls the History**

If any step feels optional, the workflow is broken.
