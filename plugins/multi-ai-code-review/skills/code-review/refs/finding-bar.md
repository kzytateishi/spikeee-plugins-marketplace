# The Bar for Every Finding

The goal is **inline-reviewer sharpness** (the kind GitHub Copilot produces): a small set
of findings that each point at an exact line and carry a concrete fix — not a broad wall
of generic advice. This bar applies to **both** `code-review` (reported findings) and
`refactor` (the "Needs Review" hand-off). Before reporting anything, it must clear all
four gates:

1. **Located** — cite `file:line` (a single line or a tight range). A finding you cannot
   anchor to a specific line does not ship.
2. **Concretely fixable** — include the minimal fix as a suggested diff (see the output
   format). If you cannot write the fix, you do not understand the issue well enough to
   report it yet — investigate or drop it.
3. **Confident** — tag each finding `confidence: high | medium`. Report only what you can
   defend by pointing at the code. **Drop anything below medium.** Do not pad the report
   with "might be", "consider possibly", or speculative concerns.
4. **Material** — it must change correctness, security, data integrity, spec-conformance,
   or violate an explicit project convention (CLAUDE.md / AGENTS.md). **Do not report pure
   style or taste** (formatting, subjective naming, "could be cleaner") — that is the
   linter's job and it only dilutes the signal.

## Fewer, sharper wins

Ten precise, line-anchored findings with fixes beat forty generic observations. A reviewer
who lists everything teaches the author to ignore the list. When in doubt, **cut it**.

- One finding = one location + one fix. Do not bundle three issues into one bullet.
- No "general suggestions" / "things to keep in mind" section. If it has no line, it is
  not a finding.
- Do not restate what the code does. State only what is **wrong** and the **fix**.
- If you find nothing above the bar in a category, say "clear" and move on. An honest
  short review is sharper than a padded long one.
