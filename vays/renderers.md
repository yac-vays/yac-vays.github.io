---
parent: VAYS
nav_order: 3
---

# Renderers

VAYS turns each form input into a React component using a *renderer*
selected from the JSON-Schema/UI-Schema. Most fields use the right
renderer automatically; for the rest, the spec author can request a
specific one via [`vays_options.renderer`](../yac/specs/file/schema.md#keyword-vays_optionsrenderer).

This page is the authoritative list of bundled renderers, their
selection rules and the options they understand.

## How Selection Works

VAYS uses [JSON Forms'](https://jsonforms.io) *tester* mechanism.  Each
renderer ships with a tester returning a numeric *rank*; the
highest-ranking match wins.  In practice:

  - **Implicit selection.** Each renderer's tester inspects the
    JSON-Schema (`type`, `format`, `enum`, ...) and the UI-Schema and
    returns a rank if it can render the field.  The default renderers
    (text, number, boolean, date, enum, ...) all use this path.
  - **Explicit selection.** Set `vays_options.renderer: <name>` in the
    YAC schema to force a specific renderer.  The bundled "special"
    renderers below are *only* selected explicitly; without the
    `renderer` option they stay out of the way.

If no custom renderer matches, VAYS falls back to JSON Forms'
`material-renderers`.

## Common Options

These options are honoured by most renderers (set on the JSON-Schema
subschema as part of `vays_options`):

| Option                    | Type      | Description |
|:--------------------------|:----------|:------------|
| `initial`                 | `any`     | Pre-filled value used as a placeholder. The value is *not* persisted unless the user interacts with the field. |
| `initial_editable`        | `boolean` | If `true`, the `initial` value is loaded as actual data (the user edits it) instead of being shown as a placeholder. |
| `renderer`                | `string`  | Name of an explicit renderer (see below). |
| `renderer_options`        | `object`  | Renderer-specific options (see the per-renderer tables below). |

The `title`, `description` and `default` JSON-Schema keywords are also
honoured everywhere, with `title` and `description` rendered as Markdown.

## Default Renderers (implicit)

These are picked automatically based on the JSON-Schema. You normally
don't have to think about them.

| Renderer                | Used for                                                | Notes |
|:------------------------|:--------------------------------------------------------|:------|
| `TextControl`           | `string` properties (no special format).                | Falls back to a single-line text input. |
| `NumberControl`         | `number` / `integer` properties.                        | Honours `minimum`, `maximum`, `exclusiveMinimum/Maximum`, `multipleOf`. |
| `BooleanControlRenderer`| `boolean` properties.                                   | Renders as a checkbox. |
| `DateControl`           | `string` with `format: date`.                           | See [Date options](#daterenderer-options). |
| `EnumControl`           | Properties with `enum`.                                 | Plain dropdown. |
| `OneOfEnumControl`      | Properties using `oneOf` for a fixed set of values.     | Dropdown with descriptive labels. |
| `MultipleChoiceRenderer`| `array` of strings constrained by `enum` / `oneOf`.     | Multi-select dropdown. |
| `VoidControl`           | Subschemas the form should not render.                  | Used internally to silently skip fields. |
| `ArrayControlRenderer`  | `array` of primitive items.                             | List with add/remove buttons. |
| `ArrayLayoutRenderer`   | `array` of objects (nested object array).               | Card-style nested layout. |
| `CategorizationLayout`  | Top-level grouping into categories (tabs).              | Driven by [`vays_category`](../yac/specs/file/schema.md#keyword-vays_category). |
| `GroupLayout`           | Sub-grouping inside a category (panel).                 | Driven by [`vays_group`](../yac/specs/file/schema.md#keyword-vays_group). |
| `NestedObjectRenderer`  | `object` properties used as rows in nested layouts.     | Used together with `ArrayLayoutRenderer`. |

### `DateControl` options

| Option              | Type     | Description |
|:--------------------|:---------|:------------|
| `enable_range.from` | `string` | Earliest selectable date (`YYYY-MM-DD`). |
| `enable_range.to`   | `string` | Latest selectable date (`YYYY-MM-DD`). |
| `disable_range.from`| `string` | Start of a disabled range (inclusive). |
| `disable_range.to`  | `string` | End of a disabled range (inclusive). |

## Special Renderers (explicit)

These renderers are *only* picked when the YAC spec explicitly requests
them via `vays_options.renderer: <name>`. Each row tells you the required
JSON-Schema shape and the options it accepts.

| `renderer` value   | JSON-Schema requirement              | Purpose |
|:-------------------|:-------------------------------------|:--------|
| `info_box`         | `type: string` (or untyped)          | Render a read-only info box (uses `title` / `description`, no input). |
| `password`         | `type: string` (or untyped)          | Password input that hashes the value before sending. |
| `ssh_key`          | `type: string` (or untyped)          | Multi-line SSH-key input with file-paste support. |
| `text_area`        | `type: string` (or untyped)          | Multi-line text area instead of a single-line input. |
| `mac_address`      | `type: string` (or untyped)          | Text input with MAC-address-aware formatting. |
| `list_as_string`   | `type: string` (or untyped)          | Edit a list of items, but persist as a separator-joined string. |
| `big_string_list`  | `type: array, items: { type: string }` | High-performance list editor for long arrays of strings. |
| `multi_checkbox`   | `type: array` with `oneOf` items     | Render the choices as a column of checkboxes instead of a multi-select. |

### `password` options

The password is hashed in the browser **before** being sent over the
wire (and before it lands in the YAML file).

| Option              | Type     | Default          | Description |
|:--------------------|:---------|:-----------------|:------------|
| `save_password_as`  | `enum`   | `crypt-sha-512`  | One of `crypt-sha-512` (Unix-`crypt`-style SHA-512 hash, like `/etc/shadow`) or `plaintext`. Only use `plaintext` if the storing system genuinely needs the cleartext value. |

### `list_as_string` options

| Option       | Type     | Default | Description |
|:-------------|:---------|:--------|:------------|
| `separator`  | `string` | `,`     | Separator inserted between items when joining the list back into a string. |

### `text_area` options

| Option   | Type      | Default | Description |
|:---------|:----------|:--------|:------------|
| `rows`   | `integer` | -       | Visible row count of the text area. |

### `info_box`, `ssh_key`, `mac_address`, `big_string_list`, `multi_checkbox`

No renderer-specific options beyond the [common options](#common-options).

## Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    welcome:
      title: Welcome
      description: |
        Please fill in **all** fields. See the [docs](https://example.com)
        if anything is unclear.
      vays_category: General
      type: string
      vays_options:
        renderer: info_box

    password:
      title: Password
      vays_category: Account
      type: string
      vays_options:
        renderer: password
        save_password_as: crypt-sha-512

    ssh_key:
      title: SSH Public Key
      vays_category: Account
      type: string
      pattern: '^(ssh-rsa|ssh-ed25519|ecdsa-sha2-[a-z0-9-]+) [A-Za-z0-9+/=]+( .+)?$'
      vays_options:
        renderer: ssh_key

    notes:
      title: Notes
      vays_category: General
      type: string
      vays_options:
        renderer: text_area
        rows: 8

    aliases:
      title: Aliases (comma-separated)
      vays_category: General
      type: string
      vays_options:
        renderer: list_as_string
        separator: ','

    tags:
      title: Tags
      vays_category: General
      type: array
      items:
        type: string
      vays_options:
        renderer: big_string_list
```
{% endraw %}

## Adding a Custom Renderer

VAYS renderers are JSON-Forms renderer/tester pairs registered in
`src/renderers/index.tsx`.  To add a new one:

  1. Create a `MyRenderer.tsx` exporting a renderer component plus a
     `MyRendererTester` built with `rankWith(rank, isCustomRenderer('my_name'))`.
  2. Add it to the relevant index file (`control/`, `combined/`,
     `layout/` or `control/special/`).
  3. Reference it from the YAC spec via
     `vays_options.renderer: my_name`.

See the [renderer source tree](https://github.com/yac-vays/vays/tree/main/src/renderers)
and its [README](https://github.com/yac-vays/vays/blob/main/src/renderers/README.md)
for conventions and helper utilities (`isCustomRenderer`,
`isUntypedStringInput`, ...).
