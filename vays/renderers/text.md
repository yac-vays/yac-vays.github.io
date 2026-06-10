---
parent: Renderers
grand_parent: VAYS
nav_order: 1
---

# Renderer `text`

The default renderer for `type: string` properties — a single-line
text input. It is also picked for *untyped* schemas that declare a
`pattern`, which lets you keep a field shape-driven without fixing the
type up front.

It is implicit (no `vays_options.renderer` required) — and is the
lowest-ranked match for strings, so any explicit
[string-based renderer](index.md#explicit-renderers) wins automatically
when its tester also matches.

## Configuration

| Keyword                                       | Effect |
|:----------------------------------------------|:-------|
| `type`                                        | Must be `string` (or omitted, with a `pattern`) for the tester to match. |
| `pattern`                                     | Non-matching values are reported inline as an error. |
| `minLength` / `maxLength`                     | Too-short / too-long values are reported inline as an error. |
| `vays_options.renderer_options.send_trivial`  | When `true`, the field stores `""` instead of clearing the value when the input is empty. Use it only when the schema or downstream system genuinely distinguishes `""` from "missing". Default `false`. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    name:
      title: Name
      description: Lower-case identifier. Letters, digits, dots and dashes only.
      vays_category: General
      type: string
      pattern: "^[a-z0-9.-]+$"
      minLength: 3
      maxLength: 32
```
{% endraw %}
