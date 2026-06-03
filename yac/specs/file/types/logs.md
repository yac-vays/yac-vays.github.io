---
parent: Section types
nav_order: 2
---

# Field `logs`

`logs` is a per-[type](types.md) field that surfaces external log or status
information for an entity (e.g. an install log or a health check) so it can be
shown in the UI. It is a **list**; each entry reads from one pluggable backend
and is rendered as a separate log in the UI.

Each entry has a `name`/`title`, two display hints, a `plugin` and
plugin-specific `details`:

| Field | Description |
|:------|:------------|
| `name`, `title` | *mandatory*; identifier and UI label of the log. |
| `progress` | `true` to render the log as a progress indicator. |
| `problem` | `true` to render the log as a problem/health indicator. |
| `plugin` | *mandatory*; one of `elastic`, `file`. |
| `details` | Plugin-specific config; all string values are [j2-strings](../j2.md). |

The `details` are rendered with the templating variables listed for
`types[].logs[].details` in the [variable matrix](../j2.md#variables). In
addition, the **per-entry** fields (`time`, `message`, `progress`, `problem`)
receive a `log` variable describing the current entry — its exact shape
depends on the plugin.

## Log plugin `elastic`

Queries an Elasticsearch endpoint. The `log` variable is each hit's `_source`
dict.

| `details` field | Description |
|:----------------|:------------|
| `url` | *mandatory*, j2-string; the Elasticsearch URL (may embed credentials from `env`). |
| `ssl_verify` | Verify the TLS certificate (default `true`). |
| `query` | *mandatory*, j2-string; the query selecting this entity's entries. |
| `message` | j2-string with the `log` hit; the displayed message. |
| `time` | j2-string with the `log` hit; the entry's timestamp. |
| `progress` | j2-int with the `log` hit (when `progress: true`). |
| `problem` | j2-bool with the `log` hit (when `problem: true`). |

{% raw %}
```yaml
logs:
  - name: installation
    title: Installation
    progress: true
    plugin: elastic
    details:
      url: "https://user:{{ env.elastic_pass }}@elastic.example.com:9200/install"
      ssl_verify: true
      query: 'host.name == "{{ name }}"'
      # `log` here is the elastic hit's `_source` dict.
      message: "{{ log.event.message }}"
      time: "{{ log['@timestamp'] }}"
      progress: "{{ log.install.percent }}"
      problem: false
```
{% endraw %}

## Log plugin `file`

Reads lines from a file and parses each with a regex. The `log` variable is
the tuple of capture groups (`log[0]`, `log[1]`, ...).

| `details` field | Description |
|:----------------|:------------|
| `path` | *mandatory*, j2-string; the path of the log file. |
| `line_format` | *mandatory*; a regex (rendered as a j2-string first). Its capture groups become the `log` tuple. |
| `message` | j2-string with the `log` tuple; the displayed message. |
| `time` | j2-string with the `log` tuple; the entry's timestamp. |
| `progress` | j2-int with the `log` tuple (when `progress: true`). |
| `problem` | j2-bool with the `log` tuple (when `problem: true`). |

{% raw %}
```yaml
logs:
  - name: status
    title: Status
    problem: true
    plugin: file
    details:
      path: "/logs/{{ name }}.log"
      # `line_format` is a regex; capture groups become the `log` tuple.
      line_format: '^\[([^\]]+)\] (.*)$'
      time: "{{ log[0] }}"
      message: "{{ log[1] }}"
      problem: "{{ log[1] is regex_match('.*failed.*') }}"
```
{% endraw %}
