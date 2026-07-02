---
parent: Renderers
grand_parent: VAYS
nav_order: 7
---

# Renderer `multiple_choice`

The default renderer for arrays whose items are constrained to a
fixed `enum` or `oneOf` set: a multi-select dropdown that lets the
user pick any combination of the offered values. The renderer is
implicit.

If `uniqueItems: true`, the dropdown behaves as a set (each value can
be picked at most once); without it, the user can repeat selections.
The [`multi_checkbox`](multi_checkbox.md) renderer is an explicit
alternative when you'd rather see all choices as a column of
checkboxes.

## Configuration

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `type`                           | Must be `array` for the tester to match. |
| `items.enum`                     | One option per entry (label = the raw value). |
| `items.oneOf[i].const`           | One option per entry (label = `oneOf[i].title` or the `const`). |
| `uniqueItems`                    | When `true`, clicking an already-selected option deselects it (no duplicates). When `false`, clicking it again appends another copy. |
| `minItems` / `maxItems`          | Wrong-size selections are reported inline as an error. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    regions:
      title: Replicated To
      description: Object replicas live in every selected region.
      vays_category: Access
      type: array
      uniqueItems: true
      items:
        enum: [eu-west-1, eu-central-1, us-east-1, ap-southeast-1]
```
{% endraw %}
