---
parent: Renderers
grand_parent: VAYS
nav_order: 3
---

# Renderer `boolean`

The default renderer for `type: boolean` — a labelled checkbox. The
renderer is implicit.

## Configuration

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `type`                           | Must be `boolean` for the tester to match. |
| `vays_options.initial_editable`  | **Silently ignored** — the renderer treats the placeholder as always editable (clicking the box toggles it). |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    public:
      title: Publicly Readable
      description: |
        Allow anonymous **read** access to objects in this bucket.
        Listings remain private.
      vays_category: Access
      type: boolean
      default: false
```
{% endraw %}
