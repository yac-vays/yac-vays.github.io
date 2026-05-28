---
parent: Renderers
grand_parent: VAYS
nav_order: 8
---

# Renderer `array`

The default renderer for arrays of primitives or of "flat" objects —
i.e. arrays whose item schema doesn't itself contain a nested array
or object. Each item is rendered inline as a row with an inline
delete button; an *Add* button appends a new item using the schema's
defaults. The renderer is implicit.

For arrays of objects with nested arrays/objects, see the
[`nested_array`](nested_arrays.md) renderer, which uses a
collapsible card layout instead.

## Behaviour

  - Items render in a vertical list. Object items use
    `uischema.options.details.elements` (as emitted by YAC), so each
    item's fields are dispatched to their own renderers — including
    explicit ones like [`age_secret`](age_secret.md) or
    [`multi_checkbox`](multi_checkbox.md).
  - Primitive items render as a single Control with scope `#` (the
    item itself), picked up by whichever default renderer matches the
    item type.
  - "Add" appends an item built from JSON-Schema defaults; "Delete"
    asks for confirmation and removes the row.

## Configuration

| Keyword                                                       | Effect |
|:--------------------------------------------------------------|:-------|
| `type`                                                        | Must be `array` for the tester to match. |
| `items`                                                       | Item schema; primitives or "flat" objects only (for nested objects, see [`nested_array`](nested_arrays.md)). |
| `minItems` / `maxItems`                                       | Wrong-size lists are reported inline as an error; the renderer does **not** disable the *Add* / *Delete* buttons to enforce them. |
| `uniqueItems`                                                 | Duplicate items are reported inline as an error. |
| `vays_options.initial` / `vays_options.initial_editable`      | **Ignored** at the array level — pre-fill items through JSON-Schema `default`, or set them on the relevant item *property* (where its renderer supports them). |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    contacts:
      title: Contacts
      description: People to notify about events.
      vays_category: Notifications
      type: array
      minItems: 1
      items:
        type: object
        properties:
          email:
            title: Email
            type: string
            format: email
          role:
            title: Role
            type: string
            enum: [owner, billing, technical]
```
{% endraw %}
