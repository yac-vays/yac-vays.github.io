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

## Configuration

Select the renderer with `vays_options.renderer: text_area`.

| Keyword                                | Effect |
|:---------------------------------------|:-------|
| `type`                                 | Must be `string` (or omitted) for the tester to match. |
| `pattern`                              | Non-matching values are reported inline as an error. |
| `minLength` / `maxLength`              | Too-short / too-long values are reported inline as an error. |
| `vays_options.renderer_options.rows`   | Visible row count of the text area (integer). Without a value, the field grows to fit its content. |

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
