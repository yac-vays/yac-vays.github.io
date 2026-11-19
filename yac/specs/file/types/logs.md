---
parent: Section types
nav_order: 2
---

# Field `logs`

`logs` is a per-[type](index.md) field that surfaces log or status
information for each entity (e.g. an install log, a health check, the last
reported facts of a host or a how-to) in the status column of the entity list.
It is a **list**; each log reads from one pluggable backend (`plugin`) and is
shown by one UI renderer (`renderer`). The two are independent: any plugin can
feed any renderer.

| Field | Description |
|:------|:------------|
| `name`, `title` | *mandatory*; identifier and UI label of the log. |
| `description` | *optional*; description of the log, shown in the UI below the title of the log details (Markdown is supported). |
| `renderer` | *optional*; how the log is shown in VAYS, one of the [log renderers](../../../../vays/log_renderers/index.md): `messages` (default), `problem`, `progress`, `document`, `markdown`. |
| `options` | *optional*; options of the renderer (see the renderer's page), e.g. `{language: yaml}` for `document`. |
| `plugin` | *mandatory*; one of `elastic`, `file`, `http`, `template`. |
| `details` | Plugin-specific config; all string values are [j2-strings](../../j2.md). |

The `details` are rendered with the templating variables listed for
`types[].logs[].details` in the [variable matrix](../../j2.md#variables)
(including the entity's data as `old.data`). In addition, the **per-entry**
fields (`time`, `message`, `progress`, `problem`) receive a `log` variable
describing the current entry — its exact shape depends on the plugin.

## Entries

Each log has a list of entries, which VAYS polls while the entity list is shown
(so fetching them should be cheap). An entry has:

| Field | Description |
|:------|:------------|
| `time` | Timestamp (should be parsable as a date, may be empty). |
| `message` | The message. |
| `problem` | `true` / `false` if the plugin's `details.problem` is defined, else `null`. |
| `progress` | 0 to 100 if the plugin's `details.progress` is defined, else `null`. |
| `has_content` | Whether the log has a content document (see below). |

A field that renders to `omit` (e.g. {% raw %}`{{ log.failed | default(omit) }}`{% endraw %}) counts as
not defined: `problem` / `progress` stay `null` for that entry, texts empty.

`problem` and `progress` are generic: whatever renderer shows the log may use
them (the `problem` renderer is a health indicator from `problem`, the
`document` renderer shows a light warning symbol on `problem`, the list of
entries marks problems and shows progresses, ...).

## Content documents

A log can provide a **content document**, e.g. a YAML file or a Markdown text,
which the `document` and `markdown` renderers show in a window. Unlike the
entries it is only fetched when it is opened (it may be big), from
`GET /entity/{type}/{name}/logs/{log}/content` (same permissions as the
entries: whoever can see the entity). Plugins that can provide content: `http`
(with `details.content`) and `template`.

## Log plugin `elastic`

Queries an Elasticsearch endpoint. The `log` variable is each hit's `_source`
dict.

| `details` field | Description |
|:----------------|:------------|
| `url` | *mandatory*, j2-string; the Elasticsearch URL (may embed credentials from `env`). |
| `ssl_verify` | Verify the TLS certificate (default `true`). |
| `timeout` | Abort the request after this many seconds (default `5`). |
| `query` | *mandatory*, j2-string; the query selecting this entity's entries. |
| `message` | j2-string with the `log` hit; the displayed message. |
| `time` | j2-string with the `log` hit; the entry's timestamp. |
| `progress` | *optional*, j2-int with the `log` hit; the entry's progress (0 to 100). |
| `problem` | *optional*, j2-bool with the `log` hit; whether the entry indicates a problem. |

{% raw %}
```yaml
logs:
  - name: installation
    title: Installation
    description: |
      The OS-installation progress of this host, as reported by the
      installer (see the [runbook](https://example.com/runbook) if it
      **fails**).
    renderer: progress
    plugin: elastic
    details:
      url: "https://user:{{ env.elastic_pass }}@elastic.example.com:9200/install"
      ssl_verify: true
      query: 'host.name == "{{ name }}"'
      # `log` here is the elastic hit's `_source` dict.
      message: "{{ log.event.message }}"
      time: "{{ log['@timestamp'] }}"
      progress: "{{ log.install.percent }}"
      problem: "{{ log.install.failed }}"
```
{% endraw %}

## Log plugin `file`

Reads lines from a file and parses each with a regex. The `log` variable is
the tuple of capture groups (`log[0]`, `log[1]`, ...).

| `details` field | Description |
|:----------------|:------------|
| `path` | *mandatory*, j2-string; the path of the log file. |
| `required` | `true` to treat a missing file as an error (default `false`). |
| `encoding` | The file encoding (default `utf-8`). |
| `line_format` | A regex (rendered as a j2-string first); its capture groups become the `log` tuple (default `^(.*)$`, i.e. the whole line as `log[0]`). |
| `message` | j2-string with the `log` tuple; the displayed message. |
| `time` | j2-string with the `log` tuple; the entry's timestamp. |
| `progress` | *optional*, j2-int with the `log` tuple; the entry's progress (0 to 100). |
| `problem` | *optional*, j2-bool with the `log` tuple; whether the entry indicates a problem. |

{% raw %}
```yaml
logs:
  - name: status
    title: Status
    renderer: problem
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

## Log plugin `http`

Fetches the entries from an HTTP(S) endpoint. Written for the
[bootstrap-logs](https://gitlab.inf.ethz.ch/public-isg/bootstrap-logs) service
(`GET /logs/{host}/{service}` returning `[{"time": ..., "message": ...}, ...]`,
`GET /states/{host}/{name}` returning a document), but works with any endpoint
whose response is — or contains — a JSON list, or that returns a document.

How the response becomes the entries depends on `response`:

  - `json` (default): the body is (or contains, see `array`) a JSON list; each
    element is an entry and the `log` variable.
  - `text`: the response is **one** entry, `log` is
    `{"status": 200, "headers": {...}, "body": "..."}` (header names in
    lowercase). Use it with `method: HEAD` to get e.g. the time of a document
    from its `last-modified` header without downloading it.

| `details` field | Description |
|:----------------|:------------|
| `url` | *mandatory*, j2-string; the request URL (may embed `name`, credentials from `env`, query params, ...). |
| `method` | HTTP method (default `GET`). |
| `headers` | Map of extra request headers, j2-strings (e.g. an auth token from `env`). |
| `response` | `json` (default) or `text`, see above. |
| `array` | For `json`: dotted path to the record list inside the response body; empty (default) means the body itself is the list (e.g. `hits.events`). |
| `empty_on` | HTTP status codes that mean "no entries" instead of an error, e.g. `[404]` for a document that doesn't exist (yet). |
| `ssl_verify` | Verify the TLS certificate (default `true`). |
| `timeout` | Abort the request after this many seconds (default `5`). |
| `message` | j2-string with the `log` record; the displayed message. |
| `time` | j2-string with the `log` record; the entry's timestamp (default `{% raw %}{{ log.time }}{% endraw %}`). |
| `progress` | *optional*, j2-int with the `log` record; the entry's progress (0 to 100). |
| `problem` | *optional*, j2-bool with the `log` record; whether the entry indicates a problem. |
| `content` | *optional*, map; if set, the log has a content document, fetched with its own request when it is opened (see below). |

The `content` map takes the same request fields as the entries (`url`,
`method`, `headers`, `response`, `empty_on`, `ssl_verify`, `timeout`); unset
ones are taken from the entries' request, except `method` (default `GET`) and
`response` (default `text`). In addition:

| `details.content` field | Description |
|:------------------------|:------------|
| `text` | j2-string with the response as `log` (the record for `text`, the parsed body for `json`); the document (default `{% raw %}{{ log.body }}{% endraw %}`). |
| `time` | j2-string like `text`; the document's timestamp (default empty). |

A log of messages:

{% raw %}
```yaml
logs:
  - name: install
    title: Install
    renderer: progress
    plugin: http
    details:
      url: "https://logs.example.com/logs/{{ name }}/install?limit=10"
      headers:
        Authorization: "Bearer {{ env.logs_token }}"
      # `log` here is each record from the returned JSON array.
      time: "{{ log.time }}"
      message: "{{ log.message }}"
      progress: '{{ 100 if log.message == "Installer finished" else (25 if log.message == "Installer started" else 0) }}'
```
{% endraw %}

A document (the Ansible facts a host reported last): the entries only ask for
the headers (`HEAD`), the document itself is downloaded when it is opened, and
facts older than 14 days get a warning.

{% raw %}
```yaml
logs:
  - name: facts
    title: Facts
    description: The Ansible facts of the last `ansible-pull` run.
    renderer: document
    options:
      language: yaml
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

## Log plugin `template`

Nothing is fetched: the entry and the content document are rendered from the
`details`, with all variables of the [variable matrix](../../j2.md#variables)
(e.g. the entity's data as `old.data`). Use it with the `markdown` renderer for
per-entity how-tos ("how to connect to this bucket"). The log has **one**
entry — or none if `content` renders to an empty string, so the template can
decide per entity whether there is something to show.

| `details` field | Description |
|:----------------|:------------|
| `content` | *mandatory*, j2-string; the document. |
| `message` | j2-string; the message of the entry, e.g. a one-line summary (default empty). |
| `time` | j2-string; the timestamp of the entry (default empty). |
| `progress` | *optional*, j2-int; the entry's progress (0 to 100). |
| `problem` | *optional*, j2-bool; whether the entry indicates a problem. |

{% raw %}
````yaml
logs:
  - name: howto
    title: How to connect
    renderer: markdown
    plugin: template
    details:
      message: Endpoint, bucket name and an example
      # e.g. warn while the bucket has no quota
      problem: "{{ (old.data.quota | default(0)) == 0 }}"
      content: |
        ## Using bucket `{{ name }}`

        | Endpoint | `https://s3.example.com` |
        |:---------|:-------------------------|
        | Bucket   | `{{ name }}`             |

        ```bash
        aws s3 ls s3://{{ name }} --endpoint-url https://s3.example.com
        ```
````
{% endraw %}

{: .warning }
Everything a `template` renders is shown to everyone who can see the entity:
don't put secrets from `env` into it.

## Upgrading from `progress` / `problem` flags

Older specs selected the display with `progress: true` / `problem: true` on the
log. These are replaced by the renderers of the same name (YAC refuses specs
that still use them): use `renderer: progress` / `renderer: problem`, and make
sure the plugin's `details.progress` / `details.problem` are defined (they used
to default to `0` / `false`).
