---
parent: File
nav_order: 2
---

# Section `repo`

The `repo` section configures the repository plugin and its connection
details, plus the per-entity-type file paths.

{% raw %}
```yaml
repo:
  plugin: git_direct      # plugin name (defaults to "git_direct")
  connection:             # plugin connection config
    url: ...
    branch: ...
  details:                # per-entity-type paths
    type_a: "path/{{ name }}.yml"
```
{% endraw %}

All values in `repo` are **static** — changes require a pod/container
restart.

Every string in `repo` is a [j2-string](../j2.md): `plugin`, `connection.*`
and `details.*` are all rendered. `plugin` and `connection.*` are
rendered **once at process startup** with the `env`-only scope (handy
for pulling secrets out of env vars). `details.*` is rendered
**per-call** inside the repo plugin and has access to `name` (and any
plugin-specific variables). See [Templating](../j2.md) for the full
variable matrix.

## Key `plugin`

The repository plugin to use. Built-in plugins:

  - `git_direct` (default): every worker clones the repo into
    `/repo/{pid}` and reads/writes files directly. Recommended for most
    deployments. Mount a tmpfs at `/repo` so the working copies live in
    memory. Every sync fetches the branch and resets the working copy to it,
    so the working copy always equals the remote: local leftovers (e.g. of
    a crashed write) are discarded and a rewritten (force-pushed) branch is
    followed.
  - `git_redis`: layered on top of `git_direct`. One pod syncs with the
    remote per TTL window and publishes a snapshot of the working tree
    into Redis; every other read across every pod is served from Redis
    with no git I/O. Reduces cold-path latency for `GET /entity/{type}` on
    large repos (10k+ entities) and shares state across pods. Writes
    still flow through `git_direct` (sync → modify → push) and publish
    a new snapshot on scope exit, so the next read sees the new commit
    immediately. External commits (pushed to git outside YAC) are
    picked up after `max_age_seconds`. Requires a reachable Redis
    instance; on Redis failure, individual requests fall back to
    `git_direct` rather than erroring.

## Key `connection` (plugins: `git_direct`, `git_redis`)

| Key                       | Type      | Default                          | Description |
|:--------------------------|:----------|:---------------------------------|:------------|
| `url`                     | `string`  | `""` (**required**)              | HTTPS or SSH URL to the git repo. Prefer HTTPS: only then can YAC tell a git server in maintenance (HTTP 503) apart from other failures, see [Remote outages](#remote-outages). |
| `branch`                  | `string`  | `main`                           | The branch to work on. |
| `ssh_key_file`            | `string`  | `/home/yac/.ssh/id_rsa`          | Path to the private key file (SSH URLs only). |
| `ssh_known_hosts_file`    | `string`  | `/home/yac/.ssh/known_hosts`     | Path to the known hosts file (SSH URLs only). |
| `dirty_max_age`           | `integer` | `0`                              | Acceptable age (in minutes) of the last git fetch where a dirty read will not update the data again. With `git_redis`, a dirty read accepts a snapshot of this age or of `max_age_seconds`, whichever is longer. |

## Key `connection` (plugin: `git_redis`)

`git_redis` accepts every `git_direct` key above (it delegates writes
and the on-pod working tree to `git_direct`) plus the Redis-specific
keys below.

| Key                | Type      | Default              | Description |
|:-------------------|:----------|:---------------------|:------------|
| `redis_url`        | `string`  | `""` (**required**)  | URL of the Redis instance, e.g. `redis://host:6379/0`. The scheme must be `redis://`, `rediss://` or `unix://` (checked at startup). |
| `max_age_seconds`  | `integer` | `300`                | How long a snapshot may serve reads before a refresh is triggered. While one pod refreshes, the others keep serving the existing snapshot; requests only wait when there is no snapshot yet. |
| `grace_seconds`    | `integer` | `60`                 | How long a replaced snapshot is kept so in-flight readers can finish; then it expires. |
| `pull_lock_ttl`    | `integer` | `120`                | TTL of the cross-pod refresh mutex. Must comfortably exceed the worst-case time of a refresh (waiting for local writers, the git sync and the upload of the snapshot). |


{: .note}
`max_age_seconds` is the **staleness budget** for external commits
(anything pushed to git outside YAC). YAC-initiated writes always
refresh the snapshot immediately on writer-scope exit.

{: .warning}
The Redis key namespace is global (`latest`, `synced`, `pull_lock`,
`snap:{hash}`). Use a dedicated Redis instance — or at minimum a
dedicated logical database (`/0`, `/1`, ...) — per YAC deployment.

{: .note}
Older YAC versions stored a snapshot as `ready:{hash}`, `paths:{hash}` and
`data:{hash}:{path}` keys. These are no longer read and never expire: flush
the Redis database once when upgrading from such a version.

## Remote outages

Both plugins are built to survive the git server being down, so that a
remote outage never takes YAC down with it:

  - **Reads keep working.** When a sync fails because the remote is
    unreachable (DNS, connection and transport failures, HTTP 5xx or
    429), the local checkout
    is kept and reads are answered from it. Such responses carry the
    headers `Warning: 110 yac "Data repository unreachable, serving last
    known state"` and `X-YAC-Repo-Synced: <unix time of the last
    successful sync>`, so clients (VAYS shows a banner) can tell the data
    may be outdated. `git_redis` likewise keeps serving its published
    snapshot.
  - **Writes fail with `503`.** A commit cannot be pushed without the
    remote, so create/edit/delete/actions answer `503 Service Unavailable`
    with a `{title, message}` body. The same happens for reads while no
    checkout exists yet (e.g. the pod started during the outage — it does
    start, and recovers by itself once the remote is back).
  - **Maintenance is recognised.** A git server that answers HTTP `503`
    (what e.g. GitLab does in maintenance mode) yields the title *Data
    Repository in Maintenance* instead of *Data Repository Unavailable*.
    This requires an **HTTPS** remote: over SSH git does not see the
    server's status code, so a maintenance window looks like any other
    unreachable remote (the regular *Unavailable* flow, still without
    losing the checkout).
  - **Backoff.** After a failed remote operation, reads do not retry the
    remote for 10 seconds but answer from the checkout right away; writes
    always retry.
  - Failures that are *not* an outage (e.g. bad credentials) answer `500`;
    the checkout is kept. A corrupt checkout is replaced by a fresh clone,
    which is built next to it and only then swapped in.
  - A push that is rejected because another YAC instance pushed in between
    is repeated once on top of the new state. If the same entity was
    changed, the request answers `409`.

See `GET /status` in the [installation guide](../../install.md#health-checks-and-monitoring)
for how to observe this.

## Key `details` (plugins: `git_direct`, `git_redis`)

A mapping from entity-type name to a [Jinja2](../j2.md) template that
resolves to the YAML file path **inside the repository** for an entity of
that type.

Each template:

  - **must** contain the variable `name` exactly once (used to
    find/list/create the entity's YAML file); the variable `env` is not
    available here;
  - **must not** contain a literal `*` (it is reserved as a glob marker
    when listing entities);
  - never matches a `/` with `name`: for {% raw %}`animals/{{ name }}.yml`{% endraw %}, the
    file `animals/old/rex.yml` is not an entity;
  - is rendered with the entity's name to obtain the file path on read,
    write, copy, link, rename and delete.

The path is interpreted relative to the repository root (no leading `/`).
Path components that escape the repository (e.g. via `..`) are rejected
at runtime.

Symlinks in the repository are only followed inside of it: an entity file (or
a directory on the way to it) that links out of the repository is not read,
and YAC never writes through a symlink.

### Limits of the entity YAML

An entity document must be a mapping (`key: value`) or empty. A list or a
scalar at the top level is rejected as invalid YAML (`400`); a stored file
of that kind reads as an entity without data.

YAML anchors and aliases (including merge keys, `<<: *defaults`) are
supported, but a document must not expand to more than 100000 nodes once its
aliases are resolved, and must not contain itself. Such a document is
rejected as invalid YAML (`400`) — a few lines of nested aliases would
otherwise expand to gigabytes of data.

## Example

{% raw %}
```yaml
repo:
  plugin: git_direct
  connection:
    url: https://user:pass@git.example.com/my/repo.git
    branch: main
  details:
    animal: "animals/{{ name }}.yml"
    cage:   "cages/{{ name }}/cage.yml"
```
{% endraw %}

With this configuration, an entity of type `animal` named `rex` is stored
at `animals/rex.yml`, and a `cage` named `c1` at `cages/c1/cage.yml`.

## Example with secret injection

Read a deploy-token (or any other secret) from the `YAC_ENV__GIT_TOKEN`
env var instead of hard-coding it in the specs file:

{% raw %}
```yaml
repo:
  plugin: git_direct
  connection:
    url: "https://yac:{{ env.git_token }}@git.example.com/my/repo.git"
    branch: main
  details:
    animal: "animals/{{ name }}.yml"
```
{% endraw %}

## Example with `git_redis`

When deployed via the Helm chart with `redis.enabled: true`, the chart
sets `YAC_ENV__REDIS_URL` on the YAC pod so the specs file can
reference it as `env.redis_url`:

{% raw %}
```yaml
repo:
  plugin: git_redis
  connection:
    url: "https://yac:{{ env.git_token }}@git.example.com/my/repo.git"
    branch: main
    redis_url: "{{ env.redis_url }}"
    max_age_seconds: 300
    grace_seconds: 60
    pull_lock_ttl: 120
  details:
    animal: "animals/{{ name }}.yml"
```
{% endraw %}

{: .important}
Every entity type defined under [`types`](types/index.md) must have a
corresponding entry in `repo.details`.
