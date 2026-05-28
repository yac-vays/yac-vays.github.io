---
parent: Renderers
grand_parent: VAYS
nav_order: 10
---

# Renderer `info_box`

Renders a **read-only block of text** instead of an input field. Use
it to inject documentation, warnings, instructions or links directly
into the form — somewhere users will actually see them.

The renderer displays only the field's `title` and `description`,
both rendered as Markdown. The underlying value is never read or
written; the field exists purely as a vehicle for the description
text.

Since the field isn't meant to carry data, mark it
`yac_optional: true` and `not: {}` so YAC doesn't allow data in the
YAML for that property.

## Configuration

Select the renderer with `vays_options.renderer: info_box`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `string` (or omitted) for the tester to match. |
| `title`                                                  | Shown as the box heading (Markdown). |
| `description`                                            | Shown as the box body (Markdown, multi-line). |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — the renderer never reads or writes form data. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    welcome:
      title: Before You Begin
      description: |
        Please fill in **all** fields. Need help? See the
        [internal docs](https://example.com/buckets) or ping
        `#storage-help` on Slack.
      vays_category: General
      yac_optional: true
      not: {}
      vays_options:
        renderer: info_box
```
{% endraw %}
