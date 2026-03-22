# Design Spec Extraction

For each section, extract and keep the exact values you will implement before writing code.
Do not approximate values by eye.

## Extract these values

### Layout

- width
- height
- min-width and min-height when present
- padding
- gap
- direction
- alignment and justification
- wrapping behavior
- max-width when the frame behaves like a fixed canvas

### Typography

- font family
- font size
- font weight
- line height
- letter spacing
- text alignment
- text transform when present

### Visual styles

- text color
- background color
- border width, style, and color
- border radius, including per-corner values
- box shadow
- opacity
- gradients

### Spacing

- section spacing
- margin-like distances implied by wrappers
- internal padding
- nested gap values

## Extraction process

1. Start with `get_design_context`.
2. Read the exact values from the returned context.
3. If any value is unclear, use `get_metadata` on the relevant child node and then re-run `get_design_context` for that child if needed.
4. When variables or styles are involved, use `get_variable_defs`.
5. Record section text exactly before implementation.
6. Save the section screenshot with `get_screenshot` for later QA.

## Suggested per-section output format

```text
## Section: Hero (node-id: 123:456)

### Layout
- direction: column
- padding: 48px 24px
- gap: 32px
- max-width: 1200px

### Title
- font-size: 32px
- font-weight: 700
- line-height: 1.4
- letter-spacing: -0.02em
- color: #1A1A1A

### Notes
- exact copy captured
- screenshot saved for section QA
```

## Rules

- Never round values unless the source itself is rounded.
- Never replace an unknown value with a guessed utility class.
- Never move on to implementation before the current section has a concrete spec.
