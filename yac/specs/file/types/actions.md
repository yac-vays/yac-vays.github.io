---
parent: Section types
nav_order: 3
---

# Field `actions`

`actions` is a per-[type](index.md) field defining custom operations that go
beyond plain CRUD: they call out to an HTTP endpoint or run a shell command.
It is a **list**; each action can be triggered explicitly by the user or
**hooked** to run automatically around create/change/delete.

| Field | Default | Description |
|:------|:--------|:------------|
| `name`, `title` | *mandatory* | Identifier and UI label. |
| `description` | `''` | Markdown, shown in the confirmation dialog when `dangerous`. |
| `icon` | a default SVG | SVG used as the button icon. |
| `dangerous` | `false` | Show a confirmation dialog (with `description`) before running. |
| `perms` | `[act]` | The user needs **at least one** of these perms on the entity to run it. |
| `force` | `false` | Run automatically on the hooked operations (see below). |
| `hooks` | `[arbitrary]` | When the action may run / is forced (see below). |
| `plugin` | *mandatory* | One of `http`, `shell`. |
| `details` | `{}` | Plugin-specific config; all string values are [j2-strings](../../j2.md). |

The `details` are rendered with the templating variables listed for
`types[].actions[].details` in the [variable matrix](../../j2.md#variables).

## Hooks and `force`

`hooks` lists the moments an action attaches to:

- `arbitrary` exposes it as a standalone button
  (`POST /entity/{type}/{name}/run/{action}`).
- `{operation}:before` / `{operation}:after` attach it to the lifecycle of
  `create`, `change` and `delete` (e.g. `create:after`, `delete:before`).

Whether a hooked action actually runs depends on `force`:

- With `force: false`, hooked actions are *offered* and run only if the client
  requests them.
- With `force: true`, hooked actions run **automatically and silently** on the
  hooked operation — and their `perms` are bypassed for all hooks **except**
  `arbitrary`.

```yaml
hooks:
  - arbitrary       # standalone button
  - create:after    # also run right after a successful create
  - delete:before
```

## Action plugin `http`

Sends an HTTP request. Useful return codes can be marked as success or as
user-facing errors.

| `details` field | Default | Description |
|:----------------|:--------|:------------|
| `method` | `GET` | The HTTP method. |
| `url` | *mandatory* | j2-string; the request URL. |
| `body` | `''` | j2-string; the request body. |
| `headers` | `{}` | j2-string values. |
| `timeout` | `5` | Abort the request after this many seconds. |
| `ssl_verify` | `true` | Verify the HTTPS certificate. |
| `success` | the 2xx codes | Status codes considered successful. |
| `error` | `[]` | Status codes whose response body is returned to the user as the error message (instead of a generic 500). |

{% raw %}
```yaml
actions:
  - name: reinstall
    title: Reinstall
    description: This **wipes** the host and reinstalls it from scratch.
    dangerous: true
    perms: [act, adm]
    hooks: [arbitrary]
    plugin: http
    details:
      method: POST
      url: "https://api.example.com/hosts/{{ name }}/reinstall"
      body: '{"requested_by": "{{ user.name }}"}'
      headers:
        Content-Type: application/json
        Authorization: "Bearer {{ env.api_token }}"
      success: [200, 202]
      error: [409]   # return the response body to the user for these codes
```
{% endraw %}

## Action plugin `shell`

Runs a shell command. Return codes can be marked as success or as user-facing
errors.

| `details` field | Default | Description |
|:----------------|:--------|:------------|
| `command` | *mandatory* | j2-string; the command to run. |
| `success` | `[0]` | Return codes considered successful. |
| `error` | `[]` | Return codes whose stdout/stderr is returned to the user as the error message. |

{% raw %}
```yaml
actions:
  - name: greet
    title: Say Hello
    hooks: [create:before]
    plugin: shell
    details:
      command: 'echo "Hello {{ user.full_name }}, creating {{ name }}"'
      success: [0]   # return codes considered successful
      error: []      # return codes whose stdout/stderr is shown to the user
```
{% endraw %}
