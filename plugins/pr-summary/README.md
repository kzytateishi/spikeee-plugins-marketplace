# PR Summary — Auto-generate Pull Request Descriptions

A Claude Code skill that generates comprehensive PR descriptions from git changes.

This skill analyzes your branch's commits, diffs, and code structure to produce a ready-to-use pull request description. It automatically detects and follows your project's PR template if one exists.

## Features

- Auto-detects your project's PR template and follows its structure
- Auto-detects the base branch (main, master, develop, or remote HEAD)
- Analyzes commits, diffs, new files, updated files, deleted files, and test coverage
- Produces a complete PR description without project-specific assumptions
- Falls back to a sensible default template when no project template is found

## Requirements

- Claude Code with skill support
- A git repository with at least one commit on the current branch

## Installation

Add the marketplace and install:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install pr-summary@spikeee-plugins
```

Or load directly from a local clone:

```bash
claude --plugin-dir /path/to/pr-summary
```

## Usage

```text
/pr-summary
```

Run this after completing your implementation, before creating a PR. The skill will analyze the current branch and generate a PR description you can review and use.

You can also pass arguments to specify a branch or additional context:

```text
/pr-summary feature/my-branch
```

## Configuration

No configuration is needed. The skill automatically adapts to your project's PR template and branch conventions.

## License

MIT
