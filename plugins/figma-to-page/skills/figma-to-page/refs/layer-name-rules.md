# Layer Name Rules

## Source of layer names

- In `get_design_context`, inspect `data-name`.
- In `get_metadata`, inspect `name`.

## Interpretation rules

### `<html-element>`

Examples: `<div>`, `<main>`, `<section>`, `<aside>`, `<h2>`, `<ul>`

- Implement that layer using the named HTML element.
- Even if a different element looks more idiomatic, prefer the explicit Figma instruction when it is clearly intentional.

### `local / ComponentName`

- Create a local component dedicated to the page or implementation unit.
- Put it in a separate file near the base implementation file.

### `ui / ComponentName`

- Reuse an existing component from the repository UI library.
- Search the project for the matching component by name and use it from its existing location.

## Decision rule

If layer naming conflicts with a generic frontend refactor, prefer the Figma naming and structure unless doing so would make the implementation incorrect.
