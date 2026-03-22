# figma-to-flutter

A Claude Code plugin that implements Flutter screens and widgets from Figma designs with design-perfect accuracy.

## How it works

```text
/figma-to-flutter [figma-url]
    │
    │  Phase 1 — Reconnaissance
    │  Resolve Figma node, scan project structure
    │
    │  Phase 2 — Extraction
    │  Pull exact values from Figma MCP (layout, typography, colors,
    │  spacing, decoration) and record a per-section spec
    │
    │  Phase 3 — Implementation
    │  Translate spec values 1-to-1 into Flutter widgets,
    │  section by section
    │
    │  Phase 4 — Review
    │  Static analysis + Gemini deep review to verify
    │  every value in code matches the spec
    ▼
  Design-perfect Flutter code
```

The key difference from manual implementation: **no value is ever guessed**. Every padding, font size, color, border radius, and shadow comes directly from Figma MCP data.

## Requirements

- **Figma MCP** — provides `get_design_context`, `get_screenshot`, `get_metadata`, `get_variable_defs`, `get_code_connect_map`
- **Flutter project** — any architecture, any state management
- **Gemini MCP** (recommended) — enables post-implementation review with extended thinking

## Installation

Add the marketplace and install:

```text
/plugin marketplace add kzytateishi/spikeee-plugins-marketplace
/plugin install figma-to-flutter@spikeee-plugins
```

Or load directly from a local clone:

```bash
claude --plugin-dir /path/to/figma-to-flutter
```

## What the review catches

Phase 4 uses Gemini with extended thinking for two review passes:

1. **Spec-code consistency** — compares every numerical value in the implementation against the Figma-extracted spec. Flags rounding, approximation, or guessed values.
2. **Code quality** — widget tree efficiency, unnecessary rebuilds, const usage, widget lifecycle, accessibility, and project convention adherence.

## License

MIT
