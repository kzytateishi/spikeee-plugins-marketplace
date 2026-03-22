# Multi-AI Code Review

A Claude Code skill that runs comprehensive code reviews, leveraging any available AI MCP servers for multi-perspective analysis.

## Features

- **Dynamic MCP Detection** — automatically detects available AI MCP servers and adapts the review workflow. Works with any AI-powered MCP server, or standalone with Claude only.
- **Parallel AI Review** — dispatches reviews to available AI engines simultaneously for faster, multi-perspective analysis.
- **Quality Gates** — auto-detects project toolchain and runs tests, linting, and type checks.
- **Convention Compliance** — checks changes against CLAUDE.md and AGENTS.md project conventions.
- **Structured Reports** — generates a consistent review report covering architecture, security, quality, and testing.

## Installation

Add the marketplace and install:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install multi-ai-code-review@spikeee-plugins
```

Or load directly from a local clone:

```bash
claude --plugin-dir /path/to/multi-ai-code-review
```

## Usage

```text
/multi-ai-code-review
```

Run from any feature branch to get a comprehensive code review comparing your changes against the base branch.

## MCP Integration

This skill dynamically detects available AI MCP servers and dispatches review tasks to them. Each available server is assigned a review perspective (e.g., architecture, security, quality). If no AI MCP servers are available, Claude performs the full review independently. All modes produce the same structured report format.

## Privacy & Security

When AI MCP servers are available, this skill sends code diffs and file contents to those external services for review. Be aware that:

- **Code is sent externally**: Your code changes are transmitted to whichever MCP servers are configured in your environment.
- **Opt-in only**: MCP servers must be explicitly configured by you; nothing is sent without your setup.
- **No MCP required**: The skill works fully with Claude alone if no MCP servers are available.
- **Review your MCP configuration**: Ensure you trust the MCP servers configured in your environment before running reviews on sensitive code.

## License

MIT
