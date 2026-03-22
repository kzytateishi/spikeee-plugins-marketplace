---
name: specify
description: >
  Generate a specification document and task list for a feature, bugfix,
  improvement, spike, or chore. Automatically detects the scenario type,
  analyzes existing codebase patterns, and produces an actionable spec
  (order.md) with a task checklist (todo.md). Use before starting implementation.
effort: max
argument-hint: <feature-description>
disable-model-invocation: true
---

# Specify

Generate a specification for `$ARGUMENTS`.

## Phase 1: Scenario Detection

Analyze the user's request and classify into one of 5 scenarios:

1. **New Feature** — keywords: "new", "add", "create", "build", "introduce"
   → Use [refs/template-new-feature.md](refs/template-new-feature.md)

2. **Improvement** — keywords: "improve", "optimize", "enhance", "update", "refine"
   → Use [refs/template-improvement.md](refs/template-improvement.md)

3. **Bugfix** — keywords: "bug", "error", "fix", "broken", "crash", "failing"
   → Use [refs/template-bugfix.md](refs/template-bugfix.md)

4. **Spike (Technical Investigation)** — keywords: "investigate", "PoC", "spike", "research", "evaluate"
   → Use [refs/template-spike.md](refs/template-spike.md)
   — Also triggered by explicit prefix: `/specify spike: ...`

5. **Chore (Technical Work)** — keywords: "refactor", "migrate", "dependency", "chore", "upgrade", "infrastructure"
   → Use [refs/template-chore.md](refs/template-chore.md)
   — Also triggered by explicit prefix: `/specify chore: ...`

**Ambiguity handling**: If the request matches keywords from multiple scenarios (e.g., "fix and improve"), STOP immediately. Output a message asking the user to choose between the top 2 matching scenarios. Do not proceed to Phase 2 until the user replies.

**No match**: If no keywords match any scenario, ask the user which scenario best fits their request.

## Phase 2: Codebase Analysis

Before writing the specification, thoroughly analyze the existing codebase:

### a. Project Environment Detection

- **Language & Framework**: detect from package manager and build configuration files
- **Project conventions**: read CLAUDE.md, AGENTS.md if present
- **Architecture docs**: search for docs/ or documentation/ directories

### b. Related Code Investigation

- Search for files and symbols related to the feature using Grep and Glob
- If a symbolic analysis MCP is available, use it for deeper code navigation (find symbol, find references, etc.)
- Identify similar implementation patterns already in the codebase
- Check database schema/migration files if the feature involves data
- Check API routing files if the feature involves endpoints

### c. External Documentation

- If a documentation MCP is available, search for related specifications
- If a library documentation MCP is available and new libraries are involved, query latest documentation
- Check any docs/ directory in the project for existing documentation

## Phase 3: Specification Generation

Using the scenario-appropriate template from refs/, generate the specification:

- Fill in all applicable sections based on codebase analysis
- Follow existing implementation patterns discovered in Phase 2
- Only write implementable content — do not guess or speculate
- Include concrete file paths based on the project's actual structure
- Reference similar existing implementations as examples

## Phase 4: Task List Generation

Create a todo.md with actionable implementation tasks:

- Tasks should be ordered by implementation dependency (what needs to be built first)
- Each task should be specific enough to implement independently
- Include test tasks alongside implementation tasks
- Use `- [ ]` checkbox format

## Output

Create two files:

1. `order.md` — Specification and implementation guide
2. `todo.md` — Task checklist in `- [ ]` format

Derive `{feature-name}` as a concise kebab-case identifier from `$ARGUMENTS` (e.g., "user authentication flow" → `user-auth-flow`). Default output directory is `specs/{feature-name}/`. If the user specifies a different directory, use that instead.

## Rules

- Follow existing codebase patterns in the specification
- Only include implementable, verified content
- Do not guess or speculate about implementation details
- Always create both order.md and todo.md
- Reference actual file paths from the project

## Next Steps (optional)

- `/implement {feature-name}` to start implementation
