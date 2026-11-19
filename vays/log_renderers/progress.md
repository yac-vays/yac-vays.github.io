---
parent: Log Renderers
grand_parent: VAYS
nav_order: 3
---

# Log Renderer `progress`

A **progress indicator**, e.g. for an OS installation: the symbol shows the
progress of the newest entry as percentage and bar. A click opens the entries with
their progress.

## Behaviour

  - **Symbol.** The `progress` (0 to 100) of the newest entry as percentage
    (coloured from red to green) with a bar along the bottom edge; at 100 a
    green check. If the newest entry reports a problem (`problem: true`), a
    red error sign replaces the percentage and the bar turns red (showing
    where it stopped). Without a `progress`, a grey question mark.
  - **Popover.** Like [`messages`](messages.md), with the percentage of each
    entry as the first column (a dash for entries without one).

## Uses

| Entry field | Use |
|:------------|:----|
| `progress` | The percentage and bar (newest entry) and the first column — define `details.progress` of the plugin. |
| `problem` | Optional: the error sign, markers in the list. |
| `message`, `time` | The list. |

## Options

None.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
types:
  - name: host
    logs:
      - name: install
        title: Install
        renderer: progress
        plugin: http
        details:
          url: "https://logs.example.com/logs/{{ name }}/install?limit=10"
          headers:
            Authorization: "Bearer {{ env.logs_token }}"
          # The installer reports {"text": ..., "progress": ..., "error": ...}
          message: "{{ (log.message | from_json).text }}"
          progress: "{{ (log.message | from_json).progress }}"
          problem: "{{ (log.message | from_json).error }}"
```
{% endraw %}
