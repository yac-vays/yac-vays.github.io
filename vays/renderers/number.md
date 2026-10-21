---
parent: Renderers
grand_parent: VAYS
nav_order: 2
---

# Renderer `number`

The default renderer for `type: number` and `type: integer` — an HTML
numeric input with up/down steppers. The renderer is implicit.

## Configuration

| Keyword                                  | Effect |
|:-----------------------------------------|:-------|
| `type`                                   | Must be `number` or `integer` for the tester to match. |
| `minimum` / `maximum`                    | Out-of-range values are reported inline as an error. |
| `exclusiveMinimum` / `exclusiveMaximum`  | Out-of-range values are reported inline as an error. |
| `multipleOf`                             | The step of the up/down arrows (instead of `1`). Non-multiple values are reported inline as an error (checked in decimal, see [Keyword `multipleOf`](../../yac/specs/file/schema.md#keyword-multipleof)). |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    quota:
      title: Quota (GB)
      description: Storage cap for this bucket.
      vays_category: Quotas
      type: integer
      minimum: 1
      maximum: 4096
      default: 10
```
{% endraw %}
