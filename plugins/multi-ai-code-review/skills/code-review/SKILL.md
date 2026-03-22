---
name: multi-ai-code-review
description: >
  Run a comprehensive code review using available AI tools in parallel.
  Analyzes architecture, code quality, security, and test coverage.
  Dynamically detects and leverages available MCP servers for multi-perspective review.
  Use when reviewing code changes, before merging PRs, or for quality audits.
effort: max
argument-hint: [branch-name]
disable-model-invocation: true
---

# Multi-AI Code Review

Run a comprehensive code review on `$ARGUMENTS` (or the current branch if no arguments are provided), leveraging any available AI MCP servers for multi-perspective analysis.

## Review Perspectives

All AI engines share these review perspectives:

| Perspective | Details |
|------------|---------|
| Architecture | Pattern appropriateness, SOLID principles, consistency with existing architecture |
| Quality | Readability, maintainability, duplication, complexity |
| Security | OWASP Top 10, input validation, authentication, authorization |
| Testing | Coverage gaps, coverage threshold compliance, test case sufficiency (normal/error/edge/boundary), edge cases |
| Performance | Inefficient data fetching, memory leaks, unnecessary computation, algorithmic complexity |
| Conventions | CLAUDE.md / AGENTS.md project convention compliance |

## Phase 1: Collect Changes

### Base Branch Detection

Determine the base branch automatically:

1. Run `git remote show origin` and look for the HEAD branch.
2. If that fails, check for `main`, `master`, or `develop` branches (in that order).
3. If multiple candidates exist, pick the first that exists locally or as a remote tracking branch.
4. If no base branch can be determined, ask the user.

### Gather Change Information

```bash
git branch --show-current
git diff <base>...HEAD --stat
git diff <base>...HEAD
git status --porcelain
git log --oneline <base>...HEAD
```

**Large diff handling**: If `git diff --stat` shows more than 1000 lines changed or 30+ files, ask the user whether to review all changes or focus on specific directories/files.

### Prepare Review Context

Read project conventions before starting the review:
- `CLAUDE.md` — coding conventions, style rules, project guidelines
- `AGENTS.md` — agent-specific instructions
- `docs/` directory — architecture and design documents

Prepare the following context to pass to AI engines:
- **diff**: full output of `git diff <base>...HEAD`
- **file_list**: output of `git diff --name-only <base>...HEAD`
- **commit_log**: output of `git log --oneline <base>...HEAD`
- **conventions**: contents of CLAUDE.md and AGENTS.md (if present)

## Phase 2: Parallel AI Review (Dynamic MCP Detection)

Check your available tools for AI-powered MCP servers and dispatch reviews in parallel. Below are known integrations — use any that are available, and skip those that are not.

### Codex — Architecture Review

Check if `mcp__codex__spawn_agent` is available. If so, spawn an agent in background with the diff, file list, commit log, and conventions, instructing it to review from an **architecture** perspective (design patterns, SOLID principles, architectural consistency, separation of concerns, dependencies, scalability).

### Gemini — Code Quality & Security Review

Check if `mcp__gemini__gemini-analyze-code` is available. If so, call it with the full diff, instructing it to review **code quality and security** (vulnerabilities, error handling, duplication, test coverage, performance).

### Other AI MCP Servers

If other AI-capable MCP tools are available, dispatch additional review perspectives to them.

### Output Format for All Dispatched Reviews

Request each engine to return findings in this format:
- **File**: path and line range
- **Severity**: Critical / High / Medium / Low
- **Category**: review perspective
- **Finding**: what the issue is
- **Recommendation**: how to fix it

**If no AI MCP servers are available**: Skip Phase 2. Claude performs a comprehensive review covering ALL perspectives in Phase 3.

## Phase 3: Claude Integrated Review

### a. Integrate Phase 2 Results

- Incorporate findings from any AI engines that returned results.
- **Consensus**: findings flagged by multiple engines → elevate priority.
- **Conflicts**: if engines disagree, provide Claude's own assessment with reasoning.
- **Gaps**: identify any perspectives not covered by Phase 2 and review those areas directly.

### b. Claude's Own Review

Whether or not Phase 2 ran, Claude reviews the changes with focus on:

- **Convention compliance**: verify against CLAUDE.md / AGENTS.md rules
- **Pattern consistency**: do changes follow existing codebase patterns?
- **Integration risks**: how do changes interact with the rest of the system?
- **Edge cases**: scenarios that automated tools commonly miss

### c. Auto-Detect and Run Quality Gates

Auto-detect the project's test, lint, and type-check commands from configuration files. Run each gate and report results. If a command is undetectable, ask the user. If the project does not use static typing, skip the type-check gate.

- **Non-interactive mode**: Always run commands in non-interactive mode to prevent watch-mode hangs.

### d. Coverage Threshold Check

1. Detect the project's coverage command and configured thresholds from configuration files
2. If thresholds exist, run the coverage command and compare results against thresholds
3. Flag any coverage shortfalls as **High** severity findings with specific uncovered lines/branches

## Output

Generate the review report following [refs/report-template.md](refs/report-template.md) with:

- Summary of changes and AI engines used
- Architecture review findings
- Code quality, security, and test coverage findings
- Integrated assessment with critical issues highlighted
- Quality gate results
- Prioritized action items

### Severity Levels

- **Critical**: Must fix before merge (security, data loss, breaking changes)
- **High**: Should fix before merge (bugs, convention violations)
- **Medium**: Recommended improvement (readability, maintainability)
- **Low**: Optional suggestion (style, optimization)

Print the report directly to the conversation. Do not create a file unless the user requests it.

## Notes

- Always run quality gates regardless of change size

