# spikeee-plugins-marketplace

Custom marketplace for Claude Code plugins by spikeee.

## Included plugins

- `figma-to-page` — Perfect UI implementation from Figma with Playwright QA
- `figma-to-flutter` — Design-perfect Flutter implementation from Figma designs
- `dev-workflow` — Complete development workflow from specification to PR creation
- `pr-summary` — Auto-generate pull request descriptions from git changes
- `multi-ai-code-review` — Comprehensive code review with dynamic multi-AI MCP integration

## Install

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install figma-to-page@spikeee-plugins
/plugin install figma-to-flutter@spikeee-plugins
/plugin install dev-workflow@spikeee-plugins
/plugin install pr-summary@spikeee-plugins
/plugin install multi-ai-code-review@spikeee-plugins
```

## Repository layout

```text
spikeee-plugins-marketplace/
├── .claude-plugin/
│   └── marketplace.json
├── plugins/
│   ├── dev-workflow/
│   ├── figma-to-flutter/
│   ├── figma-to-page/
│   ├── multi-ai-code-review/
│   └── pr-summary/
├── docs/
│   └── publish-checklist.md
├── .gitignore
├── LICENSE
└── README.md
```
