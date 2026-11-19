---
parent: Log Renderers
grand_parent: VAYS
nav_order: 2
---

# Log Renderer `problem`

A **health indicator**: the symbol shows whether the newest entry reports a
problem, e.g. for a health check or the last result of a backup job. A click
opens the entries like [`messages`](messages.md).

## Behaviour

  - **Symbol.** From `problem` of the newest entry: a green check for
    `false`, a red error sign for `true`, a grey question mark if the entry
    doesn't report one (`null`).
  - **Popover.** Like [`messages`](messages.md): the entries, problems marked
    with a red `!`.

## Uses

| Entry field | Use |
|:------------|:----|
| `problem` | The symbol (newest entry) and the markers in the list — define `details.problem` of the plugin. |
| `message`, `time` | The list. |

## Options

None.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
types:
  - name: host
    logs:
      - name: backup
        title: Backup
        renderer: problem
        plugin: file
        details:
          path: "/logs/backup/{{ name }}.log"
          line_format: '^\[([^\]]+)\] (.*)$'
          time: "{{ log[0] }}"
          message: "{{ log[1] }}"
          problem: "{{ log[1] is not regex_match('.*finished successfully.*') }}"
```
{% endraw %}
