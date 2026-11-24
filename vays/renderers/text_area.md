---
parent: Renderers
grand_parent: VAYS
nav_order: 13
---

# Renderer `text_area`

A multi-line text area for `type: string` properties. Use it for
fields that benefit from vertical space: free-form notes,
configuration snippets, long messages, etc.

The renderer debounces input by 1.5 s before writing to the form
data to keep schema-wide revalidation cheap on large forms.

Unlike the text renderer, clearing the field stores an empty string
(`""`) instead of removing the key.

## Configuration

Select the renderer with `vays_options.renderer: text_area`.

| Keyword                                | Effect |
|:---------------------------------------|:-------|
| `type`                                 | Must be `string` (or omitted, if a `pattern` is set) for the tester to match. |
| `pattern`                              | Non-matching values are reported inline as an error. |
| `minLength` / `maxLength`              | Too-short / too-long values are reported inline as an error. |
| `vays_options.renderer_options.rows`   | Visible row count of the text area (integer). Defaults to `6`; the field does not grow with its content. |
| `vays_options.initial` / `vays_options.initial_editable` | Behave as in the [text renderer](text.md). |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    notes:
      title: Notes
      description: |
        Internal notes about this bucket. Markdown is **not** rendered
        — what you type is what's stored.
      vays_category: General
      type: string
      vays_options:
        renderer: text_area
        renderer_options:
          rows: 8
```
{% endraw %}
