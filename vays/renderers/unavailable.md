---
parent: Renderers
grand_parent: VAYS
nav_order: 19
---

# Renderer `unavailable`

Renders a field as a non-interactive, error-styled box stating that **no valid
value is available**, instead of a normal (or broken/empty) input. Selected
explicitly with `vays_options.renderer: unavailable`.

{: .note}
YAC selects this renderer **automatically** for a required field whose
`enum`/`oneOf`/`anyOf` rendered to an **empty** list (e.g. a per-user `oneOf`
built with `to_consts(...)` that filtered down to nothing). In that case YAC
also makes the field unsatisfiable (`not: {}`), so the operation can never
validate and **Save is blocked** — see
[empty enumerations](../../yac/specs/file/schema.md#keywords-enum-oneof-anyof).

The renderer itself is purely presentational: it shows the message box. Whether
the field is actually invalid depends on the schema — the empty-enum path adds
`not: {}` for you; if you select the renderer by hand, add your own constraint
(or leave it informational).

## What the user sees

The field's `title` as a label, then a box containing the message from
`vays_options.renderer_options.unavailable_msg` (rendered as markdown). The
message is kept separate from the field's `description` on purpose, since the
`description` is also shown in the normal, populated case.

## Configuration

Select the renderer with `vays_options.renderer: unavailable`.

| Keyword                                              | Effect |
|:-----------------------------------------------------|:-------|
| `title`                                              | Shown as the field label above the box. |
| `vays_options.renderer_options.unavailable_msg`      | The markdown message shown in the box. Falls back to a generic note if unset. |

## [Specs](../../yac/specs/index.md) Example

A required group picked from a per-user list. When the list is empty, YAC
auto-applies this renderer (and `not: {}`) and shows `unavailable_msg`; while
the list is non-empty the field is a normal dropdown showing its `description`:

{% raw %}
```yaml
schema:
  type: object
  properties:

    group:
      title: Group
      vays_category: General
      type: string
      description: The group this entity belongs to.
      # Renders to `oneOf: []` when the lookup yields nothing → YAC makes the
      # field unavailable and shows `unavailable_msg`.
      oneOf: "{{ my_groups(user.name) | to_consts }}"
      vays_options:
        renderer_options:
          unavailable_msg: |
            You are not a member of any eligible group — please contact your
            administrator to be added, then reload this page.
```
{% endraw %}

{: .warning}
Any sibling expression that indexes the same list (e.g. a
{% raw %}`default: "{{ … | first }}"`){% endraw %} throws on an empty list
while the schema is being rendered, which fails the whole schema before the
empty-enum handling can run. Guard such expressions with
[`omit`](../../yac/specs/file/schema.md#keyword-omit),
e.g. `... | first | default(omit)`.
