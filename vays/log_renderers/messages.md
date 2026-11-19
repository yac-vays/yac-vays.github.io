---
parent: Log Renderers
grand_parent: VAYS
nav_order: 1
---

# Log Renderer `messages`

The default renderer: a **list of messages**, e.g. an install log or the
history of changes made by an external system. The symbol only says that
there are entries; a click opens a popover with them, newest first.

## Behaviour

  - **Symbol.** A neutral (grey) list symbol, independent of the entries'
    content; the newest message is the tooltip.
  - **Popover.** Title and description of the log, then each entry with its
    message and time ("5 min ago", the exact time on hover). Entries with
    `problem: true` are marked with a red `!`; if an entry reports a
    `progress`, the percentages are shown as the first column.

## Uses

| Entry field | Use |
|:------------|:----|
| `message`, `time` | The list. |
| `problem`, `progress` | Optional markers in the list. |

## Options

None.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
types:
  - name: host
    logs:
      - name: runtime
        title: Runtime
        description: The last `ansible-pull` runs of this host.
        # renderer: messages  (the default)
        plugin: http
        details:
          url: "https://logs.example.com/logs/{{ name }}/runtime?limit=10"
          headers:
            Authorization: "Bearer {{ env.logs_token }}"
          message: "{{ log.message }}"
```
{% endraw %}
