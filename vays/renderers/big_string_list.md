---
parent: Renderers
grand_parent: VAYS
nav_order: 16
---

# Renderer `big_string_list`

A list editor optimised for long arrays of strings (hundreds of
entries). The user adds items through a single input field, edits
existing entries by clicking them, and removes them with a per-item
delete button — without the form re-validating on every keystroke
across the whole list.

Compared to JSON Forms' default array control, the renderer:

  - Batches changes through a debounced callback (800 ms) when
    adding items, so rapid input doesn't flood the form's validation
    pipeline.
  - Renders the list as a single bordered chip-list rather than a
    stack of full-width inputs, which scales better visually.
  - Keeps existing items click-to-edit inline; pressing *Enter* in
    the input field appends a new one.

## Configuration

Select the renderer with `vays_options.renderer: big_string_list`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `array` for the tester to match. |
| `items.type`                                             | Items are treated as strings (other primitive types are untested). |
| `items.pattern`                                          | Non-matching items are reported inline as an error. |
| `minItems` / `maxItems`                                  | Wrong-size lists are reported inline as an error. |
| `uniqueItems`                                            | Duplicate values are reported inline as an error. |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — pre-fill the list through JSON-Schema `default` instead. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    tags:
      title: Tags
      description: |
        Free-form tags applied to this bucket. Press *Enter* in the
        input field to add a tag.
      vays_category: General
      type: array
      items:
        type: string
        pattern: '^[a-zA-Z0-9.-]+$'
      uniqueItems: true
      vays_options:
        renderer: big_string_list
```
{% endraw %}
