# dev-workflow

A Claude Code plugin providing a complete development workflow — from specification to PR creation.

## Workflow

```text
/specify [feature description]
    │  Generates specification (order.md) + task list (todo.md)
    │  Auto-detects scenario: new feature / improvement / bugfix / spike / chore
    ▼
/implement [feature-name]
    │  Implements tasks from todo.md sequentially
    │  Supports TDD or standard workflow
    ▼
/refactor
    │  Iterative refactoring (3+ rounds)
    │  Structure → Readability → Conventions → until no improvements
    ▼
/code-review
    │  Multi-perspective review (leverages available AI MCPs)
    │  Quality gates (tests, lint, type checking)
    ▼
/pr-summary
    │  Auto-generates PR description from changes
    │  Detects and follows project's PR template
    ▼
  PR ready
```

Or run the full workflow end-to-end:

```text
/dev-workflow [feature description]
```

This executes all steps in order, pausing after each for confirmation.

## Skills

| Skill | Description |
|-------|-------------|
| **dev-workflow** | Full workflow: specify → implement → refactor → code-review → pr-summary |
| **specify** | Generate spec + task list with auto scenario detection |
| **implement** | Spec-driven implementation with TDD support |
| **refactor** | Iterative refactoring with quality verification |
| **code-review** | Multi-AI review with dynamic MCP integration |
| **pr-summary** | PR description auto-generation |

## Installation

Add the marketplace and install:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install dev-workflow@spikeee-plugins
```

Or load directly from a local clone:

```bash
claude --plugin-dir /path/to/dev-workflow
```

## MCP Integration (Optional)

The code-review skill dynamically detects available AI MCP servers and dispatches review tasks in parallel for multi-perspective analysis. Without MCP servers, Claude performs all review perspectives independently.

When MCP servers are used, code diffs are sent to those external services. Ensure you trust the MCP servers configured in your environment.

## License

MIT
