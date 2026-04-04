---
name: dev-workflow
description: >
  Run the full development workflow end-to-end: specify → implement → refactor
  → code-review → pr-summary. Pauses after specify for user confirmation
  before proceeding. Pass a feature description to start from specification,
  or a feature name to resume from implementation.
effort: max
argument-hint: <feature-description>
disable-model-invocation: true
---

# Dev Workflow — Full Workflow

Run the complete development workflow for `$ARGUMENTS`.

## Execution Mode

Run all steps sequentially. By default, **pause after Step 1 (specify) and present the spec to the user for confirmation** before proceeding to implementation. Steps 2–5 run automatically, stopping only on gate failures.

Skip the specify confirmation only when the user explicitly requests it (e.g., "確認不要", "skip confirmation", "そのまま進めて").

## Workflow

Each step MUST pass its gate condition before proceeding to the next step.

```text
Step 1: /specify     → Generate specification + task list
  GATE: user confirmation (default, skippable)
Step 2: /implement   → Implement tasks from the specification
  GATE: all tests, lint, and type checks pass
Step 3: /refactor    → Iterative refactoring (3+ rounds)
  GATE: all tests, lint, and type checks pass
Step 4: /code-review → Multi-perspective code review
  GATE: no Critical or High findings remain
Step 5: /pr-summary  → Generate PR description
```

### Gate Failure Handling

If a gate fails, STOP the workflow and fix the issues before proceeding:

- **Step 1 confirmation gate**: present the spec summary and ask the user to approve, request changes, or abort. If the user requests changes, revise the spec and ask again. Do NOT proceed to Step 2 until the user explicitly approves.
- **Step 2/3 gate failure**: debug and fix until all tests, lint, and type checks pass
- **Step 4 gate failure**: fix all Critical and High findings, then re-run `/code-review` to verify. Do NOT proceed to Step 5 with unresolved Critical or High findings.

## Step 1: Specify

Generate a specification document and task list for the feature described in `$ARGUMENTS`.

Read and follow the full process defined in [specify/SKILL.md](../specify/SKILL.md).

### User Confirmation

After generating the spec, present a summary to the user:

- Detected scenario type
- Key design decisions
- Scope (number of tasks, affected files)
- Link to `order.md` and `todo.md`

Then ask the user to approve, request changes, or abort before proceeding.

## Step 2: Implement

Implement all tasks from the specification created in Step 1.

Read and follow the full process defined in [implement/SKILL.md](../implement/SKILL.md). Use the feature name derived from Step 1.

## Step 3: Refactor

Run iterative refactoring on all changed files.

Read and follow the full process defined in [refactor/SKILL.md](../refactor/SKILL.md).

## Step 4: Code Review

Run a comprehensive code review on the current branch.

Read and follow the full process defined in [code-review/SKILL.md](../code-review/SKILL.md).

## Step 5: PR Summary

Generate a pull request description from the changes.

Read and follow the full process defined in [pr-summary/SKILL.md](../pr-summary/SKILL.md).

## Completion

After all steps are done (or the user stops), summarize what was completed:

- Steps executed
- Steps skipped
- Key outputs (spec location, review findings, PR description)
