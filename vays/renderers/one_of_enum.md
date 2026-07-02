---
parent: Renderers
grand_parent: VAYS
nav_order: 6
---

# Renderer `one_of_enum`

The default renderer for properties declared as `oneOf` with `const`
values: a single-select dropdown that shows a human-friendly label
per entry while storing the underlying `const` value. The renderer is
implicit.

This renderer ranks above the plain [`enum`](enum.md) renderer, so it
wins automatically whenever the schema uses `oneOf` with `const` /
`title` pairs.

## Configuration

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `oneOf`                          | Each entry becomes a dropdown option. Each entry must declare a `const`. |
| `oneOf[i].const`                 | The value stored when this entry is selected. |
| `oneOf[i].title`                 | The label shown to the user (falls back to the `const` if missing). |
| `oneOf[i].yac_if`                | YAC-side conditional that hides the entry when the expression is false. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    visibility:
      title: Visibility
      description: Who can list objects in this bucket?
      vays_category: Access
      type: string
      oneOf:
        - { const: private, title: "Private — only the owners" }
        - { const: team,    title: "Team — anyone in the project" }
        - { const: public,  title: "Public — anyone with the URL" }
      default: private
```
{% endraw %}
