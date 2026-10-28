---
parent: Renderers
grand_parent: VAYS
nav_order: 21
---

# Renderer `limit_bar`

A number input combined with the [limit](../../yac/specs/file/types/limits.md)
it counts towards, e.g. "CPU cores per OU": a bar shows the limit's whole
usage, split into the share of **this** field and everything else (the
**others**, e.g. the other entities of the same group). The end of this
field's share can be dragged to change the value, within what is left of
the limit.

```
CPU  [8 / 10 Cores]
━━━━━━━━━━━━━━━━━━━━━━━━████████●──────────   [ 2 ]
```

It works for `type: number` and `type: integer`. The values it produces
follow the field's `multipleOf` (see [Step](#step)), so whole numbers,
tens or halves all use the same renderer.

## Behaviour

  - **Which limit.** The bar shows the limit whose
    [`path`](../../yac/specs/file/types/limits.md#field-path) names this
    field. If several limits do, the **tightest** one is shown (the one
    leaving the least room for this field). Without any such limit, only
    the number input is shown (like the [`number`](number.md) renderer).
  - **Numbers.** A chip next to the label shows the total against the cap
    (e.g. `8 / 10 Cores`), green / amber (from 80 %) / red like the usage
    chips of the other fields; this field's share on the bar has the same
    color, the others are grey. Hovering (or clicking) the chip shows the
    limit's title and the details: this entity's share, the others, what is
    free (or how much the limit is exceeded) and the field's own maximum.
  - **Dragging.** Click or drag on the bar to move the end of this field's
    share; the value snaps to the [step](#step) and is committed when the mouse is
    released. The **keyboard** works too, once the handle is focused:
    arrow keys (± one step), Page Up / Down (± ten steps), Home / End
    (minimum / maximum). The number input next to the bar allows exact
    values.
  - **Bounds.** The value can be moved from the field's `minimum` (or one
    step above `exclusiveMinimum`, `0` without either) up to the room left
    in the limit. It never passes the field's own `maximum` /
    `exclusiveMaximum`, which is marked on the bar by a thin tick. A
    value that already exceeds the limit (e.g. because the others grew in
    the meantime) can stay, but only be reduced. Typing into the number
    input is not restricted; YAC still validates the limit on save.
  - **Unset values.** An unset field counts with its
    [`vays_options.initial`](../../yac/specs/file/schema.md#keyword-vays_optionsinitial)
    value (`0` without one); its share is drawn fainter. Like `initial` in
    general, nothing is written into the data until the user changes the
    value.
  - **Live.** The usage is refreshed with every validation. While a value
    is being changed, the others stay where they are until YAC reported the
    usage for the new value.

{: .important}
The split relies on the limit counting this field's value exactly like
the field holds it, and an unset field with the field's `initial` value.
So the limit's `value` expression should use the same default as
`vays_options.initial`, e.g. both from one place in the
[`context`](../../yac/specs/file/context.md).

## Step

Dragging and the keyboard move the value in steps of the field's
`multipleOf`, so every value the bar produces is valid:

| Schema                              | Step  |
|:------------------------------------|:------|
| `type: integer`                     | `1`   |
| `type: integer`, `multipleOf: 10`   | `10`  |
| `type: number`, `multipleOf: 0.5`   | `0.5` |
| `type: number` (no `multipleOf`)    | `0.1` |

An integer field only uses an integer `multipleOf` (anything else
contradicts `type: integer`; the step is `1` then).

Any decimal `multipleOf` works (e.g. `0.005`): YAC and VAYS check it in
decimal, see [Keyword `multipleOf`](../../yac/specs/file/schema.md#keyword-multipleof).

## Configuration

Select the renderer with `vays_options.renderer: limit_bar`.

| Keyword                                  | Effect |
|:-----------------------------------------|:-------|
| `type`                                   | Must be `number` or `integer` for the tester to match. |
| `multipleOf`                             | The step, see [Step](#step). |
| `minimum` / `exclusiveMinimum`           | The lowest value the bar can be dragged to. |
| `maximum` / `exclusiveMaximum`           | The highest value the bar can be dragged to; marked on the bar. |
| `vays_options.initial`                   | The value an unset field counts with (see above). |
| `vays_options.initial_editable`          | As for the [`number`](number.md) renderer (number input only). |
| `vays_options.renderer_options.unit`     | Unit shown after the total, e.g. `Cores`. |

The limit itself is configured in the type's
[`limits`](../../yac/specs/file/types/limits.md); its `path` must name
this field.

## [Specs](../../yac/specs/index.md) Example

Cores per owner, summed over all VMs of the same owner, with a default of
`2` cores for VMs that don't set a value:

{% raw %}
```yaml
types:
  - name: vm
    title: Virtual Machines
    limits:
      - title: Cores per owner
        scope: "old.data.owner == new.data.owner"
        value: "old.data.get('cores', context.default_cores)"
        max: "16"
        path: '#/cores'

context:
  default_cores: 2

schema:
  type: object
  properties:

    cores:
      title: Cores
      description: Counts towards the cores of the owner.
      vays_category: Resources
      type: number
      multipleOf: 0.5
      exclusiveMinimum: 0
      maximum: 8
      vays_options:
        renderer: limit_bar
        initial: "{{ context.default_cores }}"
        renderer_options:
          unit: Cores
```
{% endraw %}
