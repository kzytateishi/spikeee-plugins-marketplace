---
name: implement
description: >
  Start implementation based on a specification document (order.md).
  Processes tasks from todo.md sequentially. Supports TDD (Red-Green-Refactor)
  or standard implementation workflow. Auto-detects project's test framework,
  lint tools, and coding conventions. Use after /specify.
effort: max
argument-hint: <feature-name>
disable-model-invocation: true
---

# Implement

Start implementation for `$ARGUMENTS`.

## Phase 1: Read Specification

- If `$ARGUMENTS` is a directory path, read `order.md` and `todo.md` from that directory
- Otherwise, look for `specs/{feature-name}/order.md` and `specs/{feature-name}/todo.md`
- If `todo.md` exists, process tasks from top to bottom, marking each `- [x]` on completion

**If no specification is found**: STOP and ask the user to provide the spec directory path or run `/specify` first.

## Phase 2: Project Environment Detection

Detect the project's language, frameworks, and tooling to determine the correct test/lint/build commands.

### Test, Lint, and Build Commands

Auto-detect the project's test, lint, and build commands from configuration files (package managers, task runners, build tools, etc.). If undetectable, ask the user.

- **Monorepo**: If the project is a monorepo, detect commands per package/workspace rather than the entire repository.
- **Non-interactive mode**: Always run commands in non-interactive mode to prevent watch-mode hangs.

### Coverage Command and Thresholds

Detect the project's coverage command and minimum coverage thresholds:

1. Detect the coverage command and configured thresholds from the project's configuration files
2. If thresholds exist, coverage verification becomes mandatory in Phase 4

### Project Conventions

- Read `CLAUDE.md` if present
- Read `AGENTS.md` if present
- Check `docs/` directory for architecture and coding guidelines

## Phase 3: Sequential Task Execution

For each task in the task list, execute steps 1–6 **in order**. Do NOT proceed to the next task until all quality gates in step 5 pass.

### 1. Understand the task

- Read and understand the task requirements from the specification
- Check for similar existing implementations to follow as patterns

### 2. Impact analysis

Before writing code, identify existing code that depends on or is affected by the change (callers, imports, shared types, configuration). List affected files and assess risk of breakage.

### 3. Implement

Implement based on specification, following existing codebase patterns.

### 4. Write tests

Write tests covering **all** of the following categories:

- **Normal cases**: expected inputs and outputs
- **Error cases**: invalid inputs, missing data, failure scenarios
- **Edge cases**: empty values, null/undefined, type boundaries
- **Boundary values**: min/max, off-by-one, limits

### 5. Quality gates (mandatory)

Run **all** gates. Every gate MUST pass before proceeding.

- [ ] **Tests pass**: run the project's test command — include tests for affected files identified in step 2
- [ ] **Lint passes**: run the project's lint command
- [ ] **Type check passes**: run the project's type-check command (skip if not applicable)

If any gate fails, follow the debugging procedure below. Do NOT mark the task as complete or move to the next task until all gates pass.

### 6. Mark complete

Mark the task as `- [x]` in todo.md and proceed to the next task.

### Test Failure Debugging

When tests fail, follow this procedure (max 3 attempts per failure):

1. **Read the error**: identify the failing test, error message, and stack trace
2. **Diagnose**: determine if the issue is in the implementation or the test
3. **Fix**: apply a targeted fix based on the diagnosis
4. **Re-run**: verify the fix resolves the failure without breaking other tests

If any gate still fails after 3 attempts, STOP and report the issue to the user with:
- The failing gate, test name, and error message
- What was attempted
- The suspected root cause

Do NOT continue to the next task.

## Phase 4: Completion Verification

Verify the following internally before declaring implementation complete:

- [ ] All tests pass
- [ ] Lint passes
- [ ] Type checking passes (if applicable)
- [ ] Coverage meets project thresholds (if thresholds were detected in Phase 2)
- [ ] All tasks in the task list are marked `- [x]` (if a task list exists)
- [ ] Implementation matches the specification

### Coverage Verification

If coverage thresholds were detected in Phase 2:

1. Run the coverage command
2. Compare results against the project's configured thresholds
3. If coverage is below threshold, identify uncovered lines and add tests to close the gap
4. Re-run until thresholds are met

## Rules

- Always follow existing codebase patterns
- Use the project's established test/lint commands (auto-detected in Phase 2)
- Update the task list as you progress
- Do not modify tests during TDD Green phase (when using TDD)
- Read project documentation (CLAUDE.md, docs/) before starting
- If a task is unclear, reference the specification before guessing

## Next Steps (optional)

- `/refactor` for iterative refactoring
- `/code-review` for quality review
