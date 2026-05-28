---
parent: Renderers
grand_parent: VAYS
nav_order: 15
---

# Renderer `list_as_string`

Edits a list of items in the browser, but persists them as a single
**separator-joined string** in the form data. Useful when a
downstream system (an Ansible variable, an environment variable, a
CSV field, ...) expects the value as one string but the user
experience is much nicer with item-by-item editing.

There is no built-in escaping; if your items can legitimately
contain the separator, prefer a real array field with the
[`big_string_list`](big_string_list.md) renderer instead.

## Configuration

Select the renderer with `vays_options.renderer: list_as_string`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `string` (or omitted) for the tester to match. |
| `pattern`                                                | Validated against the **joined** string, not individual items. Violations are reported inline. |
| `minLength` / `maxLength`                                | Validated against the **joined** string. Violations are reported inline. |
| `vays_options.renderer_options.separator`                | Separator used to split the incoming string and join the outgoing one. Pick something the items themselves never contain. Default `,`. |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — seed the field through JSON-Schema `default` instead. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    aliases:
      title: Aliases
      description: One alias per line. Stored as a comma-separated list.
      vays_category: General
      type: string
      pattern: '^[a-zA-Z0-9 .,-]+$'
      vays_options:
        renderer: list_as_string
        renderer_options:
          separator: ","
```
{% endraw %}
