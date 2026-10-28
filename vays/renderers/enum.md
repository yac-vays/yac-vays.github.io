---
parent: Renderers
grand_parent: VAYS
nav_order: 5
---

# Renderer `enum`

The default renderer for properties with an `enum`: a single-select
dropdown showing each enum value as both the label and the stored
value. The renderer is implicit.

If you want descriptive labels independent of the stored value, use
[`one_of_enum`](one_of_enum.md) (`oneOf` with `const`) instead — it
ranks higher and wins automatically when both shapes match.

## Unset values

An optional field (not in `required`) has an extra first entry that
removes the value again. With
[`vays_options.initial`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial)
it is named after the initial entry, e.g. `Default: eu-west-1`, and stays
selected while the field is unset: so following the default and
explicitly picking the same value are distinguishable. Without `initial`
it reads `Not set`. A required field has no such entry; its `initial`
entry is preselected instead.

## Configuration

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `enum`                           | One dropdown entry per element (label = the raw value). |
| `type`                           | Optional; the tester matches on `enum` alone. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    region:
      title: Region
      vays_category: General
      type: string
      enum: [eu-west-1, eu-central-1, us-east-1]
      default: eu-west-1
```
{% endraw %}
