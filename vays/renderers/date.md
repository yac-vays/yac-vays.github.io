---
parent: Renderers
grand_parent: VAYS
nav_order: 4
---

# Renderer `date`

Picked automatically for `type: string` properties with `format: date`
(or `vays_options.format: date`).
The user gets a calendar picker; the stored value is the ISO 8601
date string (`YYYY-MM-DD`). Clearing the field removes the value. The
renderer is implicit.

`enable_range` and `disable_range` can be combined — the picker first
restricts the user to `enable_range`, then greys out any days that
also fall inside `disable_range`.

{: .warning}
A range needs **both** `from` and `to`: an `enable_range` with only one
of them disables every date. `disable_range` currently has no effect at
all (with or without `enable_range`), so do not rely on it.

## Configuration

| Keyword                                                | Effect |
|:-------------------------------------------------------|:-------|
| `type`                                                 | Must be `string` for the tester to match. |
| `format`                                               | Must be `date` for the tester to match (alternatively `vays_options.format: date`). |
| `vays_options.renderer_options.enable_range.from`      | Earliest selectable date (`YYYY-MM-DD`). |
| `vays_options.renderer_options.enable_range.to`        | Latest selectable date (`YYYY-MM-DD`). |
| `vays_options.renderer_options.disable_range.from`     | Start of a disabled range (inclusive). |
| `vays_options.renderer_options.disable_range.to`       | End of a disabled range (inclusive). |
| `vays_options.dateFormat`                              | Display format of the picked date ([flatpickr tokens](https://flatpickr.js.org/formatting/)), default `D M j, Y`. The stored value is not affected. Not available for fields inside arrays or nested objects. |
| `vays_options.initial` / `vays_options.initial_editable` | Supported, see [`vays_options.initial`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial) and [`vays_options.initial_editable`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial_editable). |

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
