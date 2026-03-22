---
name: refactor
description: >
  Run iterative refactoring on changed files (minimum 3 rounds).
  Round 1: structural improvements (DRY, method extraction).
  Round 2: readability (naming, nesting reduction).
  Round 3: convention compliance.
  Continues until no improvements found. Generates a before/after report.
  Use after implementation, before code review.
effort: max
argument-hint: [branch-name]
disable-model-invocation: true
---

# Refactor

Run iterative refactoring on files changed in `$ARGUMENTS` (or the current branch if no arguments are provided).

## Iterative Refactoring Principle

**CRITICAL: Refactoring is NOT a one-pass process.** You MUST execute at minimum 3 separate rounds. Do NOT combine rounds or skip ahead. After completing Round 3, evaluate if more rounds are needed and continue until no improvements remain.

```text
Round 1: Structural improvements (DRY, method extraction)
   ↓ run tests + lint → verify
Round 2: Readability (naming, nesting reduction)
   ↓ run tests + lint → verify
Round 3: Convention compliance (patterns, best practices)
   ↓ run tests + lint → verify
Round N: Continue if improvements remain
```

## Phase 1: Identify Target Files

### Base Branch Detection

Detect the base branch automatically:
1. `git remote show origin` → check HEAD branch
2. Check for `main`, `master`, or `develop` branches
3. If undetectable, ask the user

### Collect Changed Files

```bash
git branch --show-current
git diff --name-only {base_branch}...HEAD
git diff {base_branch}...HEAD
```

**Important**: Refactor source code and related test code together. If no related tests exist, refactor source only and note the missing test coverage.

## Phase 2: Environment Detection

Auto-detect the project's test and lint commands from configuration files. If undetectable, ask the user.

Read project conventions from CLAUDE.md and AGENTS.md if present.

- **Non-interactive mode**: Always run commands in non-interactive mode to prevent watch-mode hangs.

## Phase 3: Iterative Refactoring Rounds

### Round 1: Structural Improvements

- [ ] Eliminate duplicate code (DRY principle)
- [ ] Split long methods/functions (target ≤ 20 lines)
- [ ] Reduce deep nesting (use early returns)
- [ ] Separate responsibilities (Single Responsibility Principle)

### Round 2: Readability

- [ ] Improve variable/function/method names (clear intent)
- [ ] Remove unnecessary comments (self-evident code)
- [ ] Simplify conditional expressions
- [ ] Extract magic numbers to named constants

### Round 3: Convention Compliance

- [ ] Check against CLAUDE.md / project conventions
- [ ] Unify with existing codebase patterns
- [ ] Apply language-specific best practices

### Round N: Additional Improvements

- [ ] Issues discovered in previous rounds
- [ ] Performance optimizations
- [ ] Edge case handling

### After Each Round

1. Run the project's test command → fix any failures
2. Run the project's lint command → fix any errors
3. Verify no behavior changes occurred
4. Assess whether further improvements are possible

## Termination Criteria

Continue iterating until ALL of the following are satisfied:

- [ ] No new improvements found
- [ ] All tests pass
- [ ] Zero lint errors
- [ ] Project conventions fully met

## Priority Order

1. **Critical**: Security issues or bugs
2. **High**: Pattern violations, convention violations
3. **Medium**: Readability, maintainability issues
4. **Low**: Optimizations, style improvements

## Rules

- Do NOT change behavior — only improve structure
- Make small, incremental changes
- Always run tests after each round
- Refactor test code alongside source code
- Do not add unnecessary comments
- Reference [refs/patterns.md](refs/patterns.md) for common refactoring patterns

## Refactoring Report

After completion, generate a report following the format in [refs/report-template.md](refs/report-template.md).

## Next Steps (optional)

- `/code-review` for quality review
- `/pr-summary` for PR description
