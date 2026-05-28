---
parent: Renderers
grand_parent: VAYS
nav_order: 17
---

# Renderer `multi_checkbox`

Renders a fixed-choice multi-select as a **column of checkboxes**,
one per option. Use it instead of the default multi-select dropdown
when the choices are few enough to fit on screen and you want every
option to be visible without opening a popup.

The renderer is the array equivalent of a checkbox for a boolean
field: each enum value gets its own checkbox (title is
`oneOf[i].title` or the raw value for `enum[i]`), and ticking or
unticking it adds or removes the value from the underlying array.

Use `items.oneOf` when you want per-option titles or
[`yac_if`](../../yac/specs/file/schema.md) conditions; use
`items.enum` when the raw values themselves are the labels you want
to display.

## Configuration

Select the renderer with `vays_options.renderer: multi_checkbox`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `array` for the tester to match. |
| `uniqueItems`                                            | Must be `true` for the tester to match. |
| `items.enum`                                             | One choice per element (label = the raw value). |
| `items.oneOf[i].const`                                   | One choice per element (label = `oneOf[i].title`). |
| `minItems` / `maxItems`                                  | Wrong-size selections are reported inline as an error. |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — seed the field through JSON-Schema `default` instead. |

## [Specs](../../yac/specs/index.md) Example

A simple permissions field driven by `items.enum`:

{% raw %}
```yaml
permissions:
  title: Permissions
  vays_category: General
  type: array
  uniqueItems: true
  minItems: 1
  items:
    enum: [read, list, write, delete]
  vays_options:
    renderer: multi_checkbox
```
{% endraw %}

Same field with per-option titles via `items.oneOf` — handy when the
stored value is a short code but you want a friendly label on the
checkbox:

{% raw %}
```yaml
permissions:
  title: Permissions
  vays_category: General
  type: array
  uniqueItems: true
  minItems: 1
  items:
    oneOf:
      - { const: read,   title: Read }
      - { const: list,   title: List }
      - { const: write,  title: "Create and Modify", yac_if: "'admins' in user.token.groups" }
      - { const: delete, title: Delete, yac_if: "'admins' in user.token.groups" }
  vays_options:
    renderer: multi_checkbox
```
{% endraw %}
