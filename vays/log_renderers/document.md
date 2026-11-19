---
parent: Log Renderers
grand_parent: VAYS
nav_order: 4
---

# Log Renderer `document`

Shows a **document** of the entity — e.g. the Ansible facts a host reported
last, a generated config file or a JSON status — in a large window with syntax
highlighting and search. The document is the log's
[content document](../../yac/specs/file/types/logs.md#content-documents), so
the log needs a plugin that provides one (`http` with `details.content`, or
`template`).

## Behaviour

  - **Symbol.** A document symbol. If the newest entry reports a problem
    (`problem: true`), a small amber warning sign is added in the corner —
    deliberately lighter than the [`problem`](problem.md) renderer's red, as
    the document is still there (e.g. "older than 14 days"). The message of
    the newest entry is the tooltip.
  - **Window.** The document is fetched when the window opens (never polled)
    and shown read-only in the same editor as the YAML expert mode:
    highlighting, folding, line numbers.
      - **Search**: `Ctrl`/`Cmd`+`F` (highlights all matches, next/previous,
        case / whole word / regex).
      - **Download**: the button next to *Close* saves it as a file (see
        `filename`).
    The header shows the entity, the time of the document (or of the newest
    entry) and the newest entry's message, with a warning sign on `problem`.
    If the document cannot be loaded, the window shows why, with *Retry*.

## Uses

| Entry field | Use |
|:------------|:----|
| `has_content` | The window shows the log's content document. |
| `problem` | Optional: the warning sign. |
| `message` | Optional: tooltip and subtitle. |
| `time` | Optional: the time shown if the document has none. |

## Options

| `options` key | Effect |
|:--------------|:-------|
| `language` | Syntax highlighting: `yaml`, or `plaintext` (default). |
| `filename` | File name of the download; `{name}` is replaced with the entity name (default `<entity>-<log>.<ext>`, the extension from `language`). |

## [Specs](../../yac/specs/index.md) Example

The facts of a host, stored in
[bootstrap-logs](https://gitlab.inf.ethz.ch/public-isg/bootstrap-logs): the
entries only ask for the headers, the document is downloaded when opened, and
facts older than 14 days get the warning sign.

{% raw %}
```yaml
types:
  - name: host
    logs:
      - name: facts
        title: Facts
        description: The Ansible facts of the last `ansible-pull` run.
        renderer: document
        options:
          language: yaml
          filename: "{name}.facts.yaml"
        plugin: http
        details:
          method: HEAD
          url: "https://logs.example.com/states/{{ name }}/facts.yaml"
          headers:
            Authorization: "Bearer {{ env.logs_token }}"
          response: text
          empty_on: [404]
          time: "{{ log.headers['last-modified'] }}"
          message: "Facts of the last Ansible run"
          problem: "{{ (log.headers['last-modified'] | to_datetime('%a, %d %b %Y %H:%M:%S GMT')) < now() - timedelta(days=14) }}"
          content:
            time: "{{ log.headers['last-modified'] }}"
```
{% endraw %}
