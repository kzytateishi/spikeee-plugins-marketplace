# Code Review Report

Findings come first and are line-anchored. Keep prose minimal — every finding is located,
fixable, and material (see [finding-bar.md](finding-bar.md)). Do not include a "general
observations" section.

## Summary
- **Branch**: {branch_name}
- **Files changed**: {file_count}
- **AI engines used**: {engines_used}
- **Verdict**: APPROVE / REQUEST_CHANGES
- **Findings**: {critical_count} Critical · {high_count} High · {medium_count} Medium

## Spec ↔ Implementation
One row per factual claim in the PR description / commit messages / linked issue. A single
MISMATCH that ships wrong behavior is Critical.

| Claim | Evidence (file:line) | VERIFIED / MISMATCH |
|-------|----------------------|---------------------|
| {claim} | {path}:{line} | {result} |

## Findings
Ordered Critical → High → Medium, then by file. One finding per block. Below-Medium is not
reported.

### [Critical] {path}:{line} — {one-line problem} · {category} · confidence: {high|medium}
**Why it's wrong:** {one or two concrete sentences — what breaks, and under what input/timing}.

```diff
- {offending line}
+ {minimal fix}
```

> Repeat the block above for each finding. For a design-level issue with no safe one-line
> fix, replace the diff with **Smallest next step:** {concrete action} and say why a patch
> isn't given.

## Quality Gates
- Tests: PASS / FAIL {detail}
- Lint: PASS / FAIL {detail}
- Type check: PASS / FAIL / N/A {detail}
- Coverage: {pct} vs threshold {threshold} — {PASS/FAIL, uncovered lines}

## Action Items
- [ ] {Critical/High items, each linked to a finding above}
