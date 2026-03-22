# Design Spec Extraction

Record exact values before writing code. This spec is the contract between Figma and implementation.

## What to extract

### Layout
- direction (Row / Column)
- width, height, min/max constraints
- padding: EdgeInsets.fromLTRB(l, t, r, b)
- gap between children
- alignment (main axis, cross axis)
- wrap behavior

### Typography
- font family
- font size
- font weight (w100–w900)
- line height → record raw value AND computed `height` (lineHeight / fontSize)
- letter spacing
- text alignment
- exact text content (character-for-character)

### Color and decoration
- fill color (hex + alpha)
- border: width, color, style
- border radius (uniform or per-corner)
- box shadow: offsetX, offsetY, blur, spread, color
- gradient: type, stops, colors, angle/center
- opacity

### Spacing
- outer spacing (margin-like padding wrappers)
- inner padding
- gap values at every nesting level

## Process

1. `get_design_context` on the section node
2. Record every value above
3. Unclear or missing? → `get_metadata` on child nodes, then `get_design_context` on the specific child
4. Variables or tokens? → `get_variable_defs`
5. `get_screenshot` for visual reference

## Format

```
Section: [Name] (node-id: [id])
Layout: [direction], padding: EdgeInsets.fromLTRB([l], [t], [r], [b]), gap: [n]
[Element]: fontSize: [n], fontWeight: w[n], height: [computed], letterSpacing: [n], color: 0x[AARRGGBB]
Background: 0x[AARRGGBB], borderRadius: [n]
Shadow: offset([x], [y]), blur: [n], spread: [n], color: 0x[AARRGGBB]
```

## Rules

- Never round
- Never guess
- Never proceed to implementation without a complete spec for the current section
