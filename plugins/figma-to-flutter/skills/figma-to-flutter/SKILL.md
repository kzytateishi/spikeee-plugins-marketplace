---
name: figma-to-flutter
description: Implement a Flutter screen or widget from a Figma URL with design-perfect accuracy. Extracts exact numerical values via Figma MCP and translates them 1-to-1 into Flutter code. No guessing, no rounding, no approximation.
argument-hint: [figma-url-or-node]
compatibility: Requires Figma MCP (get_design_context, get_screenshot, get_metadata, get_variable_defs, get_code_connect_map). Requires a Flutter project. Gemini MCP recommended for post-implementation review.
---

# Figma to Flutter

Implement the Flutter screen or widget from `$ARGUMENTS`.

Every value, every color, every font weight — extracted from Figma, not guessed. This skill's entire purpose is eliminating the gap between design and implementation.

## Phase 1 — Reconnaissance

Before touching any code:

1. **Figma node resolution** — Extract file key and node id from the URL. Convert `node-id=1-2` to `1:2`. Confirm the target variant/state.
2. **Project scan** — Identify architecture pattern, theme definitions, shared widgets, routing, asset patterns, and state management in the target project.

## Phase 2 — Extraction

This is the critical phase. All implementation decisions flow from data collected here.

### For each section of the target design:

1. `get_design_context` — structure, dimensions, styles, spacing, typography, colors
2. `get_screenshot` — visual ground truth
3. `get_metadata` — when context is truncated or insufficient, drill into child nodes
4. `get_variable_defs` — when design tokens or variables are referenced
5. `get_code_connect_map` — when existing component mappings exist

### Record the spec before writing code

Follow [refs/design-spec-extraction.md](refs/design-spec-extraction.md). Every section gets a written spec with exact values. This spec becomes the contract — code must match it exactly.

```
Section: ProfileHeader (node-id: 42:891)
Layout: Column, padding: EdgeInsets.fromLTRB(16, 24, 16, 20), gap: 12
Title: fontSize: 18, fontWeight: w600, height: 1.33, letterSpacing: -0.27, color: 0xFF1A1A1A
Subtitle: fontSize: 14, fontWeight: w400, height: 1.43, color: 0xFF6B7280
Background: 0xFFF9FAFB, borderRadius: 12
```

If a value is missing or unclear — **re-fetch from Figma MCP**. Never fill in a gap with a guess.

## Phase 3 — Implementation

Work section by section. Never implement the full screen at once.

For each section:
1. Read the recorded spec
2. Translate values 1-to-1 into Flutter widgets
3. Unknown value? Stop and go back to Phase 2. Do not proceed with a guess.

### Conversion reference

**Layout:**
Auto Layout horizontal → `Row` · vertical → `Column` · Fixed frame → `SizedBox` · Padding → `EdgeInsets.fromLTRB` with exact LTRB · Gap → `SizedBox(height: N)` or `SizedBox(width: N)` · Wrap → `Wrap(spacing: N, runSpacing: N)`

**Color:**
`#RRGGBB` → `Color(0xFFRRGGBB)` · with opacity → compute exact alpha byte · Project color constant → use only when it matches exactly

**Typography:**
fontSize, fontWeight, height (= lineHeight / fontSize), letterSpacing — all from Figma. Project TextStyle → use only when every property matches. If you'd override more than one property, don't use the theme style.

**Decoration:**
borderRadius exact values · BoxShadow with exact offset, blur, spread, color · Gradient with exact stops and colors

**Assets:**
Download from Figma, verify format, register in pubspec.yaml. Never reference Figma temporary URLs. Use project's existing image/icon patterns.

**Text content:**
Character-for-character match. Full-width/half-width, punctuation, spaces, line breaks — all exact.

## Phase 4 — Review

### 4a. Static analysis

Run `flutter analyze` and `dart format --set-exit-if-changed .` to verify zero errors and consistent formatting.

### 4b. Gemini deep review

Use `mcp__gemini__gemini-analyze-code` to verify the implementation. Pass the recorded spec from Phase 2 alongside the code and ask Gemini to use extended thinking:

**Review prompt:**
> Compare the following Figma design spec against the Flutter implementation code. Use extended thinking to thoroughly verify every numerical value (colors, font sizes, font weights, line heights, letter spacing, padding, gap, border radius, shadow values) matches exactly. Flag any value that differs, any value that appears to be guessed or approximated, and any Figma structure that was incorrectly simplified. Also check that the code follows Flutter/Dart best practices and project conventions.

Fix every discrepancy found.

### 4c. Code review

Use `mcp__gemini__gemini-analyze-code` for a second pass focused on code quality:

> Review this Flutter code for: widget tree efficiency, unnecessary rebuilds, proper const usage, correct widget lifecycle, accessibility basics (semantics labels on interactive elements), and adherence to the project's existing patterns. Use extended thinking.

## Layer name conventions

See [refs/layer-name-rules.md](refs/layer-name-rules.md).

## Hard rules

- A value not in the Figma data does not go in the code
- Rounding 12.5 to 12 is a bug
- A theme style with 3 overrides is worse than an inline TextStyle
- "Looks about right" is not a verification method
- One section complete before the next begins

## Done when

- [ ] Every section has a written spec from Figma MCP data
- [ ] Every value in code traces back to the spec
- [ ] Gemini deep review found zero spec-code mismatches (or all were fixed)
- [ ] Gemini code review passed
- [ ] `flutter analyze` reports zero errors
- [ ] Remaining discrepancies (if any) are documented with reasons
