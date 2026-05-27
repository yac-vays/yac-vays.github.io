---
parent: VAYS
nav_order: 3
has_children: true
---

# Renderers

VAYS turns each form input into a React component using a *renderer*
selected from the JSON-Schema/UI-Schema. Most fields use the right
renderer automatically; for the rest, the spec author can request a
specific one via [`vays_options.renderer`](../../yac/specs/file/schema.md#keyword-vays_optionsrenderer).

This section is the authoritative reference for the bundled renderers,
their selection rules and the options they understand. Each renderer
type has its own page:

  - [Default renderers](default.md) — picked automatically from the
    JSON-Schema (`text`, `number`, `boolean`, `date`, ...).
  - [Special renderers](special.md) — opted into explicitly via
    `vays_options.renderer: <name>` (info-box, password, SSH key, ...).
  - [`age_secret` renderer](age_secret.md) — generate-once secrets that
    are AGE-encrypted in the browser before being stored.
  - [Adding a custom renderer](custom.md) — for VAYS contributors.

## How Selection Works

VAYS uses [JSON Forms'](https://jsonforms.io) *tester* mechanism. Each
renderer ships with a tester returning a numeric *rank*; the
highest-ranking match wins. In practice:

  - **Implicit selection.** Each renderer's tester inspects the
    JSON-Schema (`type`, `format`, `enum`, ...) and the UI-Schema and
    returns a rank if it can render the field. The default renderers
    (text, number, boolean, date, enum, ...) all use this path.
  - **Explicit selection.** Set `vays_options.renderer: <name>` in the
    YAC schema to force a specific renderer. The bundled "special"
    renderers are *only* selected explicitly; without the `renderer`
    option they stay out of the way.

If no custom renderer matches, VAYS falls back to JSON Forms'
`material-renderers`.

## Common Options

These options are honoured by most renderers (set on the JSON-Schema
subschema as part of `vays_options`):

| Option                    | Type      | Description |
|:--------------------------|:----------|:------------|
| `initial`                 | `any`     | Pre-filled value used as a placeholder. The value is *not* persisted unless the user interacts with the field. |
| `initial_editable`        | `boolean` | If `true`, the `initial` value is loaded as actual data (the user edits it) instead of being shown as a placeholder. |
| `renderer`                | `string`  | Name of an explicit renderer (see the per-renderer pages). |
| `renderer_options`        | `object`  | Renderer-specific options (see the per-renderer pages). |

The `title`, `description` and `default` JSON-Schema keywords are also
honoured everywhere, with `title` and `description` rendered as Markdown.
