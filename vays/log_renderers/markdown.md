---
parent: Log Renderers
grand_parent: VAYS
nav_order: 5
---

# Log Renderer `markdown`

Shows a **Markdown text** for the entity in a large window — typically a
per-entity how-to ("how to connect to this bucket", "how to use this
database") built from the entity's data with the
[`template`](../../yac/specs/file/types/logs.md#log-plugin-template) log
plugin. The text is the log's
[content document](../../yac/specs/file/types/logs.md#content-documents), so
any plugin that provides one works.

## Behaviour

  - **Symbol.** A text symbol, with a small amber warning sign if the newest
    entry reports a problem (e.g. "the bucket has no quota yet"). The message
    of the newest entry is the tooltip.
  - **Window.** The text is fetched when the window opens (never polled) and
    rendered as [GitHub-flavoured Markdown](https://github.github.com/gfm/):
    headings, lists, tables, links, code blocks, images. Inline HTML is
    allowed but sanitized (no scripts, event handlers or styles), like in all
    Markdown VAYS shows. The header shows the entity and the newest entry's
    message (with a warning sign on `problem`). If the text cannot be loaded,
    the window shows why, with *Retry*.

## Uses

| Entry field | Use |
|:------------|:----|
| `has_content` | The window shows the log's content document. |
| `problem` | Optional: the warning sign. |
| `message` | Optional: tooltip and subtitle. |

## Options

None.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
````yaml
types:
  - name: bucket
    logs:
      - name: howto
        title: How to connect
        renderer: markdown
        plugin: template
        details:
          message: Endpoint, bucket name and an example
          problem: "{{ (old.data.quota | default(0)) == 0 }}"
          content: |
            ## Using bucket `{{ name }}`

            | Endpoint | `https://s3.example.com` |
            |:---------|:-------------------------|
            | Bucket   | `{{ name }}`             |

            ```bash
            aws s3 ls s3://{{ name }} --endpoint-url https://s3.example.com
            ```

            {% if (old.data.quota | default(0)) == 0 %}
            **Note:** the bucket has no quota yet, writes will fail.
            {% endif %}
````
{% endraw %}

{: .warning }
The text is visible to everyone who can see the entity: don't render secrets
into it.
