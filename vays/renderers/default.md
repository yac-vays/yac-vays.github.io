---
parent: Renderers
grand_parent: VAYS
nav_order: 1
---

# Default Renderers

These are picked automatically based on the JSON-Schema. You normally
don't have to think about them.

| Renderer                | Used for                                                | Notes |
|:------------------------|:--------------------------------------------------------|:------|
| `TextControl`           | `string` properties (no special format).                | Falls back to a single-line text input. |
| `NumberControl`         | `number` / `integer` properties.                        | Honours `minimum`, `maximum`, `exclusiveMinimum/Maximum`, `multipleOf`. |
| `BooleanControlRenderer`| `boolean` properties.                                   | Renders as a checkbox. |
| `DateControl`           | `string` with `format: date`.                           | See [Date options](#datecontrol-options). |
| `EnumControl`           | Properties with `enum`.                                 | Plain dropdown. |
| `OneOfEnumControl`      | Properties using `oneOf` for a fixed set of values.     | Dropdown with descriptive labels. |
| `MultipleChoiceRenderer`| `array` of strings constrained by `enum` / `oneOf`.     | Multi-select dropdown. |
| `VoidControl`           | Subschemas the form should not render.                  | Used internally to silently skip fields. |
| `ArrayControlRenderer`  | `array` of primitive items.                             | List with add/remove buttons. |
| `ArrayLayoutRenderer`   | `array` of objects (nested object array).               | Card-style nested layout. |
| `CategorizationLayout`  | Top-level grouping into categories (tabs).              | Driven by [`vays_category`](../../yac/specs/file/schema.md#keyword-vays_category). |
| `GroupLayout`           | Sub-grouping inside a category (panel).                 | Driven by [`vays_group`](../../yac/specs/file/schema.md#keyword-vays_group). |
| `NestedObjectRenderer`  | `object` properties used as rows in nested layouts.     | Used together with `ArrayLayoutRenderer`. |

## `DateControl` options

| Option              | Type     | Description |
|:--------------------|:---------|:------------|
| `enable_range.from` | `string` | Earliest selectable date (`YYYY-MM-DD`). |
| `enable_range.to`   | `string` | Latest selectable date (`YYYY-MM-DD`). |
| `disable_range.from`| `string` | Start of a disabled range (inclusive). |
| `disable_range.to`  | `string` | End of a disabled range (inclusive). |
