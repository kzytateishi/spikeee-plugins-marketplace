# Figma to Page — Perfect UI Implementation with Playwright QA

A Claude Code plugin for high-fidelity Figma implementation.

This plugin turns a standalone Figma implementation skill into a shareable Claude Code plugin. It is designed for teams that want exact Figma reproduction to take priority over generic refactoring or design interpretation.

## What it enforces

- Figma visual fidelity first
- Figma structure fidelity first
- Exact copy matching
- Strict layer-name interpretation
- Section-by-section implementation
- Playwright QA before moving to the next section

## Requirements

- Claude Code with plugin support
- Figma MCP configured and reachable
- Playwright available in the target repository
- A target page, route, story, or harness that can be opened for QA

## Installation

Add the marketplace and install:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install figma-to-page@spikeee-plugins
```

Or load directly from a local clone:

```bash
claude --plugin-dir /path/to/figma-to-page
```

## Usage

```text
/figma-to-page:implement https://www.figma.com/design/<fileKey>/<fileName>?node-id=1-2
```

## Configuration

This plugin uses the following conventions by default:

- `ui / ComponentName` reuses the repository’s existing UI component library
- `local / ComponentName` becomes a separate local file near the base implementation file

Edit the skill text if your repository uses a different component path or naming convention.

## License

MIT
