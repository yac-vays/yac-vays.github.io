---
parent: VAYS
nav_order: 3
has_children: true
---

# Renderers

VAYS turns each form input into a React component using a *renderer*
selected from the JSON-Schema and UI-Schema produced by the [Schema]((../../yac/specs/file/schema.md).
Most fields get the right renderer automatically; for the rest, the spec
author opts into a specific one via [`vays_options.renderer`](../../yac/specs/file/schema.md#keyword-vays_optionsrenderer).

Renderers can have their own configuration in the schema under
[`vays_options.renderer_options`](../../yac/specs/file/schema.md#keyword-vays_optionsrenderer_options).
In addition to this, there are two common options each renderer may implement:
[`vays_options.initial`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial) and
[`vays_options.initial_editable`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial).

## How Selection Works

VAYS uses [JSON Forms'](https://jsonforms.io) *tester* mechanism. Each
renderer ships with a tester function that, given a UI-Schema element
plus the resolved JSON-Schema (both derived from YACs Schema), returns
a numeric *rank* if it can render the field. The highest-ranking match
wins; ties are broken by registration order. There are two paths to a match:

  - **Implicit selection.** The tester inspects the JSON-Schema
    (`type`, `format`, `enum`, ...) and returns a rank without needing
    anything special in the spec. The default renderers (text, number,
    boolean, date, enum, ...) all live here.
  - **Explicit selection.** Setting `vays_options.renderer: <name>` in
    the YAC schema forces a specific renderer (the tester additionally
    checks for the right `renderer` key). The "special" renderers in
    the second table below are *only* selected this way.

If no VAYS renderer matches, JSON Forms falls back to its bundled
`material-renderers`.

## Implicit Renderers

Picked automatically from the JSON-Schema:

| Renderer                                  | Used for                                                       | Since |
|:------------------------------------------|:---------------------------------------------------------------|------:|
| [`text`](text.md)                         | `type: string` (or untyped with `pattern`).                    |    v0 |
| [`number`](number.md)                     | `type: number` / `integer`.                                    |    v0 |
| [`boolean`](boolean.md)                   | `type: boolean`.                                               |    v0 |
| [`date`](date.md)                         | `type: string` with `format: date`.                            |    v0 |
| [`enum`](enum.md)                         | Properties with `enum`.                                        |    v0 |
| [`one_of_enum`](one_of_enum.md)           | Properties using `oneOf` with `const` values.                  |    v0 |
| [`multiple_choice`](multiple_choice.md)   | `type: array` of `enum` or `oneOf` items.                      |    v0 |
| [`array`](array.md)                       | Arrays of primitives or "flat" objects.                        |    v0 |
| [`nested_array`](nested_arrays.md)        | Object arrays whose items contain a nested array/object.       |    v0 |
| `categorization`                          | One tab for each [`vays_category`](../../yac/specs/file/schema.md#keyword-vays_category). |    v0 |
| `group`                                   | One box for each [`vays_group`](../../yac/specs/file/schema.md#keyword-vays_group) inside a `vays_category`. |    v0 |
| `void`                                    | Don't show everything else (to allow subschemas that are not in the form). |    v0 |

## Explicit Renderers

Selected via `vays_options.renderer: <name>`:

| Renderer                                 | Purpose                                                            | Since |
|:-----------------------------------------|:-------------------------------------------------------------------|------:|
| [`info_box`](info_box.md)                | Read-only info box (renders `title` and `description` only).       |    v0 |
| [`password`](password.md)                | Password input that hashes the value before sending.               |    v0 |
| [`ssh_key`](ssh_key.md)                  | Multi-line SSH-key input with file-paste support.                  |    v0 |
| [`text_area`](text_area.md)              | Multi-line text area instead of a single-line input.               |    v0 |
| [`mac_address`](mac_address.md)          | Auto-formats and validates MAC addresses.                          |    v0 |
| [`list_as_string`](list_as_string.md)    | Edits a list but persists as a separator-joined string.            |    v0 |
| [`big_string_list`](big_string_list.md)  | High-performance list editor for long arrays of strings.           |    v0 |
| [`multi_checkbox`](multi_checkbox.md)    | Render array-of-enum choices as a column of checkboxes.            |    v0 |
| [`age_secret`](age_secret.md)            | Generate-once secret, AGE-encrypted in the browser before storage. |    v0 |
