---
parent: Renderers
grand_parent: VAYS
nav_order: 4
---

# Renderer `date`

Picked automatically for `type: string` properties with `format: date`.
The user gets a calendar picker; the stored value is the ISO 8601
date string (`YYYY-MM-DD`). The renderer is implicit.

`enable_range` and `disable_range` can be combined — the picker first
restricts the user to `enable_range`, then greys out any days that
also fall inside `disable_range`.

## Configuration

| Keyword                                                | Effect |
|:-------------------------------------------------------|:-------|
| `type`                                                 | Must be `string` for the tester to match. |
| `format`                                               | Must be `date` for the tester to match. |
| `vays_options.renderer_options.enable_range.from`      | Earliest selectable date (`YYYY-MM-DD`). |
| `vays_options.renderer_options.enable_range.to`        | Latest selectable date (`YYYY-MM-DD`). |
| `vays_options.renderer_options.disable_range.from`     | Start of a disabled range (inclusive). |
| `vays_options.renderer_options.disable_range.to`       | End of a disabled range (inclusive). |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    valid_from:
      title: Valid From
      description: First day on which the access policy applies.
      vays_category: Validity
      type: string
      format: date
      vays_options:
        renderer_options:
          enable_range:
            from: "2025-01-01"
            to:   "2026-12-31"
          disable_range:
            from: "2025-12-23"
            to:   "2026-01-02"   # block the year-end freeze period
```
{% endraw %}
