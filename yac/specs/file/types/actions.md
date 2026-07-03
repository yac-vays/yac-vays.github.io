---
parent: Section types
nav_order: 3
---

# Field `actions`

`actions` is a per-[type](index.md) field defining custom operations that go
beyond plain CRUD: they call out to an HTTP endpoint or run a program.
It is a **list**; each action can be triggered explicitly by the user or
**hooked** to run automatically around create/edit/delete.

| Field | Default | Description |
|:------|:--------|:------------|
| `name`, `title` | *mandatory* | Identifier and UI label. |
| `description` | `''` | Markdown. Shown in the action's info popup and, when `dangerous`, in the confirmation dialog. |
| `icon` | a default SVG | SVG used as the button icon. |
| `dangerous` | `false` | Show a confirmation dialog (with `description`) before running. |
| `perms` | `[act]` | The user needs **at least one** of these perms on the entity to run it. |
| `force` | `false` | Run automatically on the hooked operations (see below). |
| `hooks` | `[arbitrary]` | When the action may run / is forced (see below). |
| `plugin` | *mandatory* | One of `http`, `exec`. |
| `details` | `{}` | Plugin-specific config; all string values are [j2-strings](../../j2.md). |

The `details` are rendered with the templating variables listed for
`types[].actions[].details` in the [variable matrix](../../j2.md#variables).

## Hooks and `force`

`hooks` lists the moments an action attaches to:

- `arbitrary` exposes it as a standalone button
  (`POST /entity/{type}/{name}/run/{action}`).
- `{operation}:before` / `{operation}:after` attach it to the lifecycle of
  `create`, `edit` and `delete` (e.g. `create:after`, `delete:before`).

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

## Action plugin `exec`

Runs a program. The command is an argv list and every
element is a j2-string rendering to exactly **one** argument, so untrusted
values (token claims like `user.full_name`, entity data) can never break out
of their argument.

| `details` field | Default | Description |
|:----------------|:--------|:------------|
| `command` | *mandatory* | Argv list (`["/path/prog", "arg", ...]`); each element a j2-string yielding one argument. Prefer an absolute program path. |
| `env` | `{}` | Extra environment variables (j2-string values), merged over a minimal base env (`PATH`, `HOME`, `HOSTNAME`, `PWD`, `LANG`). |
| `stdin` | `''` | Data piped to the process's stdin (j2-string; a template rendering to an object/list is JSON-encoded). Empty means stdin is closed. |
| `success` | `[0]` | Return codes considered successful. |
| `error` | `[]` | Return codes whose stdout/stderr is returned to the user as the error message. |
| `timeout` | `0` (none) | Seconds after which the process is killed (server-side error). |

Pick the data channel by sensitivity: **argv** is world-readable inside the
pod (`/proc/<pid>/cmdline`) — fine for plain parameters like an entity name;
**env** is readable only by same-uid processes — fine for private values;
**stdin** is readable by nobody else — use it for secrets and bulk data.

{% raw %}
```yaml
actions:
  - name: greet
    title: Say Hello
    hooks: [create:before]
    plugin: exec
    details:
      command: ["/usr/bin/greet", "--entity", "{{ name }}"]
      env:
        GREET_USER: "{{ user.full_name }}"
      stdin: "{{ new.data | tojson }}"
      success: [0]   # return codes considered successful
      error: []      # return codes whose stdout/stderr is shown to the user
      timeout: 30
```
{% endraw %}

{: .warning}
You *can* run a shell explicitly — `command: ["/bin/sh", "-c", "<script>"]` —
but **be careful**: Badly validated j2-variables (e.g. a too widely open
regular expression) might introduce a shell injection vulnerability! Hand data
to a script via `env`/`stdin` instead.

### Programs available in the image

`command[0]` must be a program that exists **inside the YAC container**. The
official image (`python:3.14-alpine` plus a few `apk` packages — see the
[`Dockerfile`](https://github.com/yac-vays/yac/blob/main/Dockerfile)) ships the
programs below. Prefer an **absolute path** (the `exec` plugin runs with a
minimal `PATH` of `/usr/local/bin:/usr/bin:/bin`).

| Program | Path | Notes |
|:--------|:-----|:------|
| Python  | `/usr/local/bin/python3` (also `python`, `python3.14`) | Python 3.14. YAC's own dependencies are importable (`httpx`, `jinja2`, `jsonschema`, `ruamel.yaml`, `ldap3`, `redis`, …), so a small `python3 -c '…'` is usually the most convenient action. |
| `pip` / `pip3` | `/usr/local/bin/pip3` | Present, but the image is read-only for the runtime user — do **not** install packages at action time. |
| POSIX shell | `/bin/sh` (also `/bin/ash`) | [BusyBox](https://busybox.net/) `ash`. BusyBox also provides `awk`, `sed`, `grep`, `wget`, `nc`, `base64`, `sha256sum`/`md5sum` and the usual coreutils (`cat`, `cut`, `tr`, `sort`, `head`/`tail`, `find`, `xargs`, `date`, `env`, …). There is **no** `bash`. |
| `git`   | `/usr/bin/git` | Full git client. |
| `curl`  | `/usr/bin/curl` (and `wcurl`) | Full curl (HTTP/2, TLS). For plain HTTP calls prefer the [`http` plugin](#action-plugin-http) instead. |
| OpenSSH client | `/usr/bin/ssh` (also `scp`, `sftp`, `ssh-keygen`) | The runtime user's home (`/home/yac/.ssh`) is writable for keys/`known_hosts`. |

{: .note}
If you need a program that is not (yet) included, you may either extend the
official YAC image to build your own version or you mount that extra program
into the container at runtime or you open a pull-request with the updated
Dockerfile on GitHub.
