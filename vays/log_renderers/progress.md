---
parent: Log Renderers
grand_parent: VAYS
nav_order: 3
---

# Log Renderer `progress`

A **progress indicator**, e.g. for an OS installation: the symbol is a ring
filled to the progress of the newest entry. A click opens the entries with
their progress.

## Behaviour

  - **Symbol.** A ring filled to `progress` (0 to 100) of the newest entry,
    with the percentage in the middle (`NA %` if it reports none). If the
    newest entry reports a problem (`problem: true`), a red exclamation mark
    replaces the percentage; with `problem: false` a faint green check is shown
    behind it.
  - **Popover.** Like [`messages`](messages.md), with the percentage of each
    entry as the first column (a dash for entries without one).

## Uses

| Entry field | Use |
|:------------|:----|
| `progress` | The ring (newest entry) and the first column — define `details.progress` of the plugin. |
| `problem` | Optional: the exclamation mark / check in the ring, markers in the list. |
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
