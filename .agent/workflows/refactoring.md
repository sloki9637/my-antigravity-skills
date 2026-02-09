---
description: Refactoring Workflow
---

# Refactoring Workflow

This workflow is designed to safely perform **refactoring that changes structure without changing behavior**.

Refactoring is **not** feature development.  
👉 **Behavior preservation** is the highest priority.

---

## Core Principles (Refactoring Only)

- **Behavior must not change**
- **Structure may change**
- **Risk must be isolated**
- **History must explain intent**

If any of these are violated, it is no longer refactoring.

---

## 1. Define Refactoring Intent (Mandatory)

> Refactoring without a clear intent is a controlled failure.

### Must be defined

- Why the structure needs to change
- Problems in the current structure
- Target design improvements
- Explicit **Non-Goals** (what must not change)

### Rules

- Intent must exist **before any code changes**
- “It looks cleaner” is not an acceptable reason

---

## 2. Register Refactoring Issue (Safety Anchor)

> Refactoring always requires an Issue.

### Execution

- Use `git-issue-formatter`
- Register the issue on GitHub or GitLab

### Mandatory rules

- Issue content **must be written in English**
- Refactoring issues must be **separate from feature issues**
- The issue becomes the reference point for:
  - Branch naming
  - Commits
  - Pull Requests
  - Review criteria

### Required Issue Sections (Refactoring-specific)

- Refactoring Background (structural problems)
- Refactoring Goal (design objectives)
- Non-Goals
- Refactoring Plan (step-by-step)
- Behavior Preservation Checklist
- Risk & Rollback Strategy

---

## 3. Create Refactoring Branch

> Refactoring must be isolated.

### Branch naming

```text
refactor/#123-short-description
```

### Rules

- Never mix with feature branches
- One branch = one refactoring intent
- If feature work appears:
  - Stop immediately
  - Split into a separate issue/branch

---

## 4. Pre-Refactoring Safety Check (Mandatory)

> Refactoring without protection is gambling.

### Before starting

- Confirm existing test coverage
- Identify critical behavior paths
- If tests are missing:
  - Add minimal characterization tests, or
  - Explicitly document the risk in the issue

### Rules

- Large structural changes without safety checks are forbidden

---

## 5. Refactoring Execution (Small & Incremental)

> Change less, more often.

### Execution rules

- Keep changes small and reviewable
- Maintain a valid build at all times
- Never commit broken or half-finished states

### Forbidden changes (unless explicitly stated)

- Behavior or spec changes
- Public API changes
- UI/UX changes

---

## 6. Self Review: Behavior Verification (Mandatory)

> Only one question matters: “Does it behave exactly the same?”

### Checklist

- Same external behavior
- Same inputs and outputs
- Same error handling
- Performance impact understood and acceptable

### If deviations are found

- Stop refactoring
- Document the impact in the issue
- Split into a feature/change issue if needed

---

## 7. Commit & Push (Intent Preservation)

> Refactoring commits exist to explain _why_, not just _what_.

### Commit rules

- Use `git-commit-formatter`
- Commit messages must:
  - Be written in English
  - Include the issue number
  - Clearly explain the refactoring intent

### Example

```text
refactor: reorganize CountdownTimer responsibilities (#123)
```

### Push

```bash
git push -u origin <branch-name>
```

---

## 8. Pull Request Creation (Refactoring Log)

> A refactoring PR is a safety report, not a change list.

### Execution

- Use `git-pr-formatter`

### PR must include

- Refactoring intent summary
- Before/after structural explanation
- Behavior preservation checklist
- Test results
- Risks and rollback plan

---

## 9. PR Review: Refactoring Review Summary (Mandatory)

> Review the design decisions, not just the diff.

### Review focus

- Behavior preservation
- Responsibility boundaries
- Complexity reduction
- Future extensibility

### Rules

- Review summary must be written in English
- “Looks fine” is not acceptable
- Design trade-offs must be explicitly documented

---

## 10. Merge & Closure

A refactoring issue may be closed only when **all** conditions are met:

- PR is merged
- Behavior preservation is confirmed
- Refactoring Review Summary exists
- Rollback strategy is documented

---

## Refactoring Failure Signals

Stop immediately if any of the following appear:

- Features quietly added
- Large changes without tests
- Vague or abstract commit messages
- “Refactor first, figure it out later” mindset
- Inability to clearly explain the outcome

👉 If this happens, it is no longer refactoring.
