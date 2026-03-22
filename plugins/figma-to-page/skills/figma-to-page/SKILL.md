---
name: figma-to-page
description: Implement a frontend page or component from a Figma URL or node using Figma MCP and Playwright QA. Use when exact visual, copy, and structural fidelity to Figma is the top priority.
argument-hint: [figma-url-or-node]
---

# Figma to Page

Implement the frontend page or component from `$ARGUMENTS`.

## Core priorities

- Exact Figma visual and structural fidelity comes first.
- Prefer the structure explicitly present in Figma over generic frontend best practices.
- Pixel drift, copy drift, and unjustified structural simplification count as defects.
- Work section by section. Do not implement the whole page first and reconcile later.

## Preconditions

- Figma MCP must be available. If `get_design_context` or `get_screenshot` are unavailable, stop and verify the connection before implementing.
- The target Figma URL or node must resolve to a concrete file key and node id.
- A local page, route, story, or test harness must exist so Playwright can verify the implementation.

## Required workflow

### 0. Investigate the target project

Before writing any code:

1. Identify the framework and styling system in use.
2. Check for existing constants, utilities, and shared components that should be reused.
3. Check for framework-specific image components and use them instead of raw `<img>` tags when available.
4. Identify the existing page/route structure and follow the same patterns for file organization.
5. Extract repeated data (FAQ items, feature lists, navigation items) into constants files following the project's existing pattern.
6. Check for existing shared assets (store badges, social icons, common logos) that can be reused across pages.
7. Identify the project's font configuration and verify Figma fonts are available.

### 1. Identify the target node

- Extract the Figma file key and node id from the URL or provided node reference.
- Convert `node-id=1-2` style URL parameters to `1:2` when a tool expects colon notation.
- For branch URLs, treat the branch key as the effective file key when required by the tooling.
- If the UI has variants or states, confirm that the selected variant or state matches the implementation target.

### 1b. Check for responsive variants

- Look for sibling frames named "Mobile", "SP", "Tablet" at the same level as the target node.
- If found, collect design context for both desktop and mobile frames.
- Implement responsive layouts using the project's breakpoint system.
- Do not ship a PC-only fixed-width implementation when a mobile frame exists in the Figma file.

### 2. Collect the design sources of truth

1. Call `get_design_context` for the target node.
2. Treat the returned code as reference material for structure, dimensions, styles, variants, and states, not as production-ready final code.
3. If the response is too large, truncated, or insufficient for structure decisions, call `get_metadata` and then re-run `get_design_context` on the necessary child nodes.
4. Call `get_screenshot` for the same node and use it as the visual ground truth.
5. When variables or style tokens are involved, call `get_variable_defs`.
6. When component reuse is possible, call `get_code_connect_map` before inventing new wrappers or duplicate components.
7. For each section you are about to implement, follow [refs/design-spec-extraction.md](refs/design-spec-extraction.md).

### 3. Interpret structure in this order

When deciding how to map Figma structure into code, use this precedence order:

1. Code Connect mappings
2. Figma layer structure
3. Layer name conventions
4. The nearest matching existing component in the codebase

Layer naming rules are defined in [refs/layer-name-rules.md](refs/layer-name-rules.md).

### 4. Prepare assets before implementation

- Extract all asset URLs from `get_design_context` responses before writing any code.
- Batch download all assets to the project's local asset directory.
- Immediately after downloading, verify each file's actual format by inspecting its content (e.g. `<svg` header, PNG magic number `\x89PNG`, JPEG `\xFF\xD8`). Rename mismatched files in batch before writing any component code — this is not a separate step to do later.
- For complex composite assets (store badges, logos with overlaid text), first check if the project already has identical assets in a shared directory. Reuse existing project assets when visually identical. If not available, export the parent Figma node as a single image rather than relying on individual vector layers.
- Create a clear local-path mapping and use only local paths in source code. Never reference Figma MCP temporary URLs in source files.
- Do not substitute Figma assets with placeholders or unrelated icon or image libraries unless the design source truly does not provide the asset.

### 5. Implement one section at a time

For each section, complete this loop before moving to the next one:

1. Extract exact structure, text, tokens, and measurements for the current section.
2. Implement only that section.
3. Run Playwright QA for that section.
4. Compare the implementation against the Figma section screenshot.
5. Fix the differences immediately.
6. Only proceed after the current section is aligned.

Do not defer fidelity fixes to the very end.

## Implementation rules

- Preserve the Figma layer structure as much as reasonably possible.
- Do not merge wrapper elements that materially affect spacing, alignment, clipping, background boundaries, or DOM structure.
- Reuse existing components where they fit without breaking fidelity.
- If an existing component is close but not exact, prefer a minimal wrapper or adapter instead of a broad redesign.
- Create `local / ...` components as separate files alongside the base implementation file.
- Reuse `ui / ...` components from the repository’s existing UI library. Search the project for the matching component by name.
- Apply basic form semantics correctly with `id`, label association (`for`/`htmlFor` depending on framework), `fieldset`, and `legend` where appropriate.
- Use semantic HTML elements: `<header>`, `<nav>`, `<footer>`, `<section>`, `<a>` for links, `<button>` for interactive elements.
- Navigation items that scroll to sections must be `<button>` elements with scroll handlers, not plain `<span>` or `<div>`.
- Store download badges and external links must be wrapped in `<a>` tags with appropriate `href`.
- Do not use `<div onClick>` where `<button>` is semantically correct.
- Always include accessibility attributes required by HTML semantics (e.g. `alt` on images, `aria-label` on icon-only buttons). Do not add redundant or decorative ARIA attributes beyond what the element semantics require.
- Match Figma text exactly: spelling, punctuation, spaces, line breaks, full-width or half-width characters, and repeated whitespace where relevant.
- Do not guess spacing, radius, border, shadow, opacity, typography, or color values.

### Wide viewport centering

- If the Figma frame has a fixed width, the implementation must center content on viewports wider than that width.
- Apply max-width and centering per section, not to the global app root, so that full-bleed backgrounds remain intact.
- For sections with absolute-positioned content, ensure the positioned container is also constrained and centered.
- Never use fixed left-offset positioning on elements that should be centered. Always wrap in a max-width centered container.
- After implementing the first section, verify at a viewport wider than the design width to catch centering issues early.

### Converting Figma fixed values to responsive CSS

Figma outputs fixed pixel values. Some must be preserved exactly, others must be converted:

**Preserve exactly** (internal sizing):
- font-size, line-height, letter-spacing
- padding, gap within components
- border-radius, border-width
- icon and image dimensions within components

**Convert to responsive** (layout positioning):
- Fixed left or right offsets from the viewport edge → relative positioning within a centered container
- Fixed gap values used to space header or nav elements across the full width → flexible distribution within a fixed-width container
- Frame-width values on sections → full-width outer element with max-width centered inner content

**Rule of thumb**: If a pixel value positions content relative to the viewport edge, it needs a centering wrapper. If it sizes content internally, preserve it exactly.

### Responsive implementation strategy

When both desktop and mobile designs exist:

1. **Same structure, different sizing** → Use the project's responsive system (media queries, utility classes, or equivalent) to adjust sizes at breakpoints. Prefer this approach to reduce code duplication.
2. **Fundamentally different layouts** (e.g. horizontal grid → vertical stack) → Use conditional visibility to render separate DOM structures per breakpoint.
3. Document which approach was chosen per section.

## Text handling

- Treat `get_design_context` and the Figma screenshot as the authoritative text sources.
- Use `get_metadata` primarily for structure and hierarchy inspection, not as the only source of truth for final copy.
- For text where line breaks or repeated spaces matter, verify the rendered DOM with strict `textContent` comparison during QA.

## Playwright QA

Follow [refs/playwright-qa.md](refs/playwright-qa.md).

### Dev server startup

- Before starting QA, verify the dev server can start successfully.
- If the default dev command fails, try alternative startup methods available in the project before giving up.
- If the dev server cannot be started at all, explicitly report this as a blocker rather than silently skipping QA.
- Never declare implementation complete without at least one visual verification.

### Mandatory viewport checks

Every section must be verified at all of the following viewport widths:

| Width | Purpose |
|-------|---------|
| Design width (e.g. 1440px) | Baseline fidelity check |
| Design width + 480px (e.g. 1920px) | Wide viewport centering check |
| Mobile design width (e.g. 393px) | Mobile layout check (when SP frame exists) |

The wide viewport check is mandatory. Most centering bugs only appear at widths larger than the design canvas. If the first section fails the wide viewport check, fix it before implementing subsequent sections — centering bugs tend to be systemic.

### First-section gate

After implementing the first section:

1. Take screenshots at all required viewport widths.
2. Verify content is centered at the wide viewport width.
3. If centering fails, fix the centering pattern before proceeding.
4. This catches systemic issues early.

### Per-section QA checklist

For each completed section, verify:

- Visual match at design width (compare with Figma screenshot)
- Content centered at wide viewport (no left-drift)
- Mobile layout correct at mobile width (when SP frame exists)
- Critical text matches Figma exactly
- Critical computed styles match (font-size, color, spacing)
- No console errors
- Each interactive state or variant verified (if applicable)

### What NOT to accept as QA

- A single full-page screenshot at design width only
- Snapshot (accessibility tree) only without visual screenshot
- Declaring "looks good" without comparing against Figma
- Skipping the wide viewport check

## Forbidden actions

- unrequested UI changes
- adding decorative details that do not exist in Figma
- reorganizing the UI into a custom component architecture that ignores the Figma structure
- summarizing, rewriting, or fixing Figma copy
- substituting non-Figma assets when the design source already provides them
- implementing the whole page first and postponing section QA until the end
- rounding or approximating Figma pixel values
- declaring QA complete based only on a full-page or scaled-down screenshot

## Completion criteria

- `get_design_context` and `get_screenshot` were used before implementation
- `get_metadata`, `get_variable_defs`, and `get_code_connect_map` were used when needed
- the implementation followed a section-by-section build, QA, and fix loop
- copy matches Figma
- visual differences were checked with Playwright
- exported asset file types were verified when assets were downloaded
- console errors are zero
- navigation links are functional (scroll-to-section or page navigation)
- external links (store badges, social icons) have correct `<a>` tags
- page metadata (title, description, OGP) is set when the target is a full page
- accordion/toggle interactions use accessible patterns (`<button>` + state)
- any remaining differences are explicitly documented with reasons

## Final report

Summarize only the essential outcomes:

- files implemented or changed
- existing components reused
- new local components created
- sections verified with Playwright
- major differences found and fixed
- remaining differences and why they remain
