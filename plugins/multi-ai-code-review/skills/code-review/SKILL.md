---
name: multi-ai-code-review
description: >
  Run a comprehensive code review using available AI tools in parallel.
  Analyzes architecture, code quality, security, and test coverage.
  Dynamically detects and leverages available MCP servers for multi-perspective review.
  Use when reviewing code changes, before merging PRs, or for quality audits.
effort: xhigh
argument-hint: [branch-name]
disable-model-invocation: true
---

# Multi-AI Code Review

Run a comprehensive code review on `$ARGUMENTS` (or the current branch if no arguments are provided), leveraging any available AI MCP servers for multi-perspective analysis.

## The bar for every finding (read first)

Every finding you report MUST clear the bar in [refs/finding-bar.md](refs/finding-bar.md):
**located** (`file:line`), **concretely fixable** (minimal suggested diff), **confident**
(`high`/`medium` only — drop the rest), and **material** (correctness / security / data /
spec / explicit convention — never pure style). Prefer **fewer, sharper findings**: ten
line-anchored issues with fixes beat forty generic observations. This bar is what makes the
review as pointed as an inline PR reviewer instead of a wall of advice.

## What the review is built around

The **spine** of this review is [refs/high-signal-checklist.md](refs/high-signal-checklist.md)
— the concrete, recurring issues that automated PR reviewers (e.g. GitHub Copilot) flag and
that generic "review for quality" prompts miss: spec↔code mismatch, enum/constant
hardcoding, nil-on-nullable, time/range boundaries, get-or-insert races, missing indexes,
XSS escaping, weak test assertions, message↔logic drift. Walking the diff against every
category of that checklist is the **primary** job (Phase 3.b), and pre-empts the post-PR
churn of fixing these one comment at a time.

The perspectives below are **secondary lenses** — a coverage net so nothing whole-cloth is
missed. They are necessary but not sufficient; do not let them turn the review into generic
advice. Any finding from a lens still has to clear the finding bar.

| Perspective | Details |
|------------|---------|
| Architecture | Pattern appropriateness, SOLID principles, consistency with existing architecture |
| Quality | Readability, maintainability, duplication, complexity |
| Security | OWASP Top 10, input validation, authentication, authorization |
| Testing | Coverage gaps, coverage threshold compliance, test case sufficiency (normal/error/edge/boundary), edge cases |
| Performance | Inefficient data fetching, memory leaks, unnecessary computation, algorithmic complexity |
| Conventions | CLAUDE.md / AGENTS.md project convention compliance |
| Consistency | PR description / commit messages / linked issue vs. the actual diff (claimed scope, definitions, DB impact, behavior all match the code) |

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

**PR description / intent (required for the Consistency perspective)**: capture the stated
intent so the review can cross-check it against the diff (checklist category A).

```bash
# If a PR already exists for this branch:
gh pr view --json title,body 2>/dev/null
```

If no PR exists yet, use the commit messages (`git log`) and the linked issue
(Linear/Jira ID in the branch name or commits) as the statement of intent. Fetch the
issue body if the tooling is available.

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
- **intent**: PR title/body (or commit messages + linked issue) — the claims to verify
- **conventions**: contents of CLAUDE.md and AGENTS.md (if present)

## Phase 2: Parallel AI Review (Dynamic MCP Detection)

Check your available tools for AI-powered MCP servers and dispatch reviews in parallel. Below are known integrations — use any that are available, and skip those that are not.

Pass the **finding bar and the high-signal checklist into every dispatched prompt**, and
require each engine to anchor findings to `file:line` with a minimal fix. An engine that
returns generic prose without locations is producing noise — re-prompt it for located
findings or discard its output.

### Codex — Architecture Review

Check if `mcp__codex__spawn_agent` is available. If so, spawn an agent in background with the diff, file list, commit log, and conventions, instructing it to review from an **architecture** perspective (design patterns, SOLID principles, architectural consistency, separation of concerns, dependencies, scalability).

### Gemini — Code Quality & Security Review

Check if `mcp__gemini__gemini-analyze-code` is available. If so, call it with the full diff, instructing it to review **code quality and security** (vulnerabilities, error handling, duplication, test coverage, performance).

### Other AI MCP Servers

If other AI-capable MCP tools are available, dispatch additional review perspectives to them.

### Output Format for All Dispatched Reviews

Request each engine to return findings in this format:
- **File**: path and line range
- **Severity**: Critical / High / Medium
- **Category**: review perspective / checklist category
- **Finding**: what is wrong (not what the code does)
- **Fix**: the minimal concrete change (a diff if possible)
- **Confidence**: high / medium

**If no AI MCP servers are available**: Skip Phase 2. Claude performs a comprehensive review covering ALL perspectives in Phase 3.

## Phase 3: Claude Integrated Review

### a. Integrate Phase 2 Results

- Incorporate findings from any AI engines that returned results — but **only those that
  clear the finding bar.** Discard located-but-immaterial nits and unlocated prose.
- **Consensus**: findings flagged by multiple engines → elevate priority.
- **Conflicts**: if engines disagree, provide Claude's own assessment with reasoning.
- **Gaps**: identify any perspectives not covered by Phase 2 and review those areas directly.

### b. Claude's Own Review — checklist-driven (the core pass)

This is the primary pass. Do it whether or not Phase 2 ran.

**Step 1 — Spec ↔ implementation table (checklist category A).** Take each factual claim in
the captured intent (PR/commit/issue) and point at the line that proves or contradicts it.
Emit the claim-by-claim table from the report template (`Claim | file:line | VERIFIED /
MISMATCH`) **before** anything else. A MISMATCH that ships wrong behavior is Critical.

**Step 2 — Walk every checklist category against the diff.** Go through
[refs/high-signal-checklist.md](refs/high-signal-checklist.md) categories B–I in order — B.
enum/constant hardcoding, C. nil/nullable, D. time & range boundaries, E. concurrency &
idempotency, F. indexes & query efficiency, G. web security & a11y, H. test rigor, I.
message/i18n↔logic. Do not skim. For each category, either emit located findings or write
"clear". Each finding gets `file:line`, severity, confidence, and a minimal-diff fix.

**Step 3 — Secondary lenses.** Sweep the perspectives table for anything the checklist
didn't cover (architecture fit, integration risks, convention compliance, edge cases the
checklist doesn't enumerate). Same bar applies — located, fixable, material, or it doesn't
ship.

### c. Auto-Detect and Run Quality Gates

Auto-detect the project's test, lint, and type-check commands from configuration files. Run each gate and report results. If a command is undetectable, ask the user. If the project does not use static typing, skip the type-check gate.

- **Non-interactive mode**: Always run commands in non-interactive mode to prevent watch-mode hangs.

### d. Coverage Threshold Check

1. Detect the project's coverage command and configured thresholds from configuration files
2. If thresholds exist, run the coverage command and compare results against thresholds
3. Flag any coverage shortfalls as **High** severity findings with specific uncovered lines/branches

## Output

Report findings the way an inline PR reviewer does — located, minimal, actionable —
following [refs/report-template.md](refs/report-template.md). Print the report directly to
the conversation. Do not create a file unless the user requests it.

Each finding is one block (outer fence shown with four backticks so the inner diff renders):

````
[Severity] path/to/file.rb:42 — one-line statement of the problem · category · confidence: high

Why it's wrong: one or two concrete sentences — what breaks, under what input/timing.

```diff
- offending line
+ minimal fix
```
````

- Lead with `file:line`. One finding = one location + one fix. The diff must be minimal —
  change only what the finding requires.
- Group findings Critical → High → Medium, then by file. **Below Medium is not reported.**
- For a design-level issue with no safe one-line fix, replace the diff with the smallest
  concrete next step and say why a patch isn't given.
- Always include the spec↔implementation table before the findings.
- No "general suggestions" / "keep in mind" section. If it has no line, it is not a finding.

### Severity Levels

- **Critical**: Must fix before merge (security, data loss, breaking change, a spec mismatch that ships wrong behavior)
- **High**: Should fix before merge (bug, convention violation, missing index on a filtered/sorted column, weak test that passes while broken)
- **Medium**: Worth fixing, with a concrete fix (maintainability, a non-blocking checklist hit). Anything below Medium is dropped, not downgraded.

## Notes

- Always run quality gates regardless of change size
