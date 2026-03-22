---
name: pr-summary
description: >
  Auto-generate a pull request description from git changes.
  Analyzes commits, diffs, and code structure to produce a comprehensive
  PR summary. Automatically detects and follows your project's
  PR template if present. Use after completing implementation,
  before creating a PR.
argument-hint: [branch-name]
disable-model-invocation: true
---

# PR Summary

Generate a pull request description from `$ARGUMENTS` (or the current branch if no arguments are provided).

## Phase 1: PR Template Detection

Search for a PR template using two Glob patterns **in parallel** (both MUST be executed):
- `**/*PULL_REQUEST_TEMPLATE*`
- `**/*pull_request_template*`

From the combined results, select the first match according to this priority order:

1. Root: `PULL_REQUEST_TEMPLATE.md` or `pull_request_template.md`
2. `.github/PULL_REQUEST_TEMPLATE.md` or `pull_request_template.md`
3. `docs/PULL_REQUEST_TEMPLATE.md` or `pull_request_template.md`
4. `.github/PULL_REQUEST_TEMPLATE/` directory (use the default template)

If no match is found, use the default template from [refs/default-template.md](refs/default-template.md).

Read the selected template. This template defines the exact sections and structure the final PR description must follow.

## Phase 2: Git Information Collection

Collect **all** git information in a **single Bash call** by running the following combined command. This is critical to minimize API round-trips and avoid rate limiting:

```bash
echo "===BRANCH===" && git rev-parse --abbrev-ref HEAD && echo "===BASE===" && git remote show origin 2>/dev/null | grep 'HEAD branch' | sed 's/.*: //' || (git branch -a --format='%(refname:short)' | grep -E '^(origin/)?(main|master|develop)$' | head -1 | sed 's|origin/||') && echo "===STATUS===" && git status --porcelain && echo "===LOG===" && git log --oneline $(git remote show origin 2>/dev/null | grep 'HEAD branch' | sed 's/.*: //' || echo main)...HEAD 2>/dev/null && echo "===STAT===" && git diff --stat $(git remote show origin 2>/dev/null | grep 'HEAD branch' | sed 's/.*: //' || echo main)...HEAD 2>/dev/null && echo "===DIFF===" && git diff $(git remote show origin 2>/dev/null | grep 'HEAD branch' | sed 's/.*: //' || echo main)...HEAD 2>/dev/null
```

Parse the output by the `===SECTION===` delimiters to extract each piece of information.

If the base branch cannot be determined from the output, ask the user which branch to compare against.

If there are uncommitted changes shown in the STATUS section, note them but focus the PR description on committed changes.

**Large diff handling**: If the diff exceeds 1000 lines or 30+ files, run a second command to process changes by directory or module rather than as a single block. Summarize at a higher level to stay within context limits.

## Phase 3: Code Analysis

Analyze the diff output collected in Phase 2. **Do NOT read individual changed files** unless the diff is ambiguous or truncated — the diff itself contains sufficient information for the PR description.

### Specification Reference

If `specs/{feature-name}/order.md` exists, read it to produce a more accurate and context-rich summary. Extract feature background, goals, and implementation decisions. Otherwise, skip this step.

### Analysis (from diff only)

From the diff and commit log collected in Phase 2, identify:
- **New files**: Purpose, new modules/classes/functions/components introduced.
- **Updated files**: What changed and why. Patterns across changes (refactoring, feature addition, bug fix). Changes to public APIs or interfaces.
- **Deleted files**: What was removed and likely reason.
- **Test files**: New/updated tests, what they cover, coverage alignment with changes.
- **Configuration and infrastructure**: Build configs, CI/CD, dependency files.

## Phase 4: PR Description Generation

Generate the PR description by filling in the template detected in Phase 1.

### Rules

- Fill in each section of the template with relevant information from Phases 2 and 3.
- **Do NOT add sections that are not in the template.** If the template has no "Impact" section, do not create one.
- **Do NOT add AI generation footers or comments** such as "Generated with Claude" or similar.
- **Do NOT change the template section order.** Preserve the exact ordering of sections.
- **Do NOT guess content.** If information is unknown, leave the section blank or write "N/A".
- **Do NOT include raw diff output.** Summarize changes in human-readable language.
- Write in clear, concise language. Use bullet points for lists of changes.
- Reference specific file paths when describing changes.
- Use commit messages as supporting context, but do not copy them verbatim.
- For the summary section, write 1-3 sentences that explain the purpose and motivation of the PR at a high level.
- For change lists, group related changes logically rather than file-by-file.
- For test plan sections, describe how the changes were tested based on test files found, or leave checkboxes for the author to fill in.
- For checklist sections, check off items verifiable from the diff (e.g., "Tests added" if test files are present).

## Output

Present the generated PR description to the user in a markdown code block so it can be easily copied. Do not wrap it in additional commentary before the code block.

After presenting the description, briefly note:
- The base branch used for comparison.
- The number of commits and files changed.
- Any uncommitted changes that were not included.

## Next Steps (optional)

- Copy the generated description when creating a PR
