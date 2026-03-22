# Layer Name Rules

## Source of layer names

- In `get_design_context`, inspect `data-name`.
- In `get_metadata`, inspect `name`.

## Interpretation rules

### `widget / WidgetName`

- Reuse an existing widget from the project's shared widget library.
- Search the project for the matching widget by name and use it from its existing location.

### `local / WidgetName`

- Create a local widget dedicated to the current screen or feature.
- Put it in a separate file within the same feature directory.

### `screen / ScreenName`

- This layer represents a full screen. Map it to a screen-level widget in the project's routing structure.

## Decision rule

If layer naming conflicts with a generic Flutter refactor, prefer the Figma naming and structure unless doing so would make the implementation incorrect.
