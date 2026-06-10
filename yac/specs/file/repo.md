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
    memory.
  - `git_redis`: layered on top of `git_direct`. One pod pulls from the
    remote per TTL window and publishes a snapshot of the working tree
    into Redis; every other read across every pod is served from Redis
    with no git I/O. Reduces cold-path latency for `GET /entity/{type}` on
    large repos (10k+ entities) and shares state across pods. Writes
    still flow through `git_direct` (pull → modify → push) and rebuild
    the snapshot on scope exit, so the next read sees the new commit
    immediately. External commits (pushed to git outside YAC) are
    picked up after `max_age_seconds`. Requires a reachable Redis
    instance; on Redis failure, individual requests fall back to
    `git_direct` rather than erroring.

## Key `connection` (plugins: `git_direct`, `git_redis`)

| Key                       | Type      | Default                          | Description |
|:--------------------------|:----------|:---------------------------------|:------------|
| `url`                     | `string`  | `""` (**required**)              | HTTPS or SSH URL to the git repo. |
| `branch`                  | `string`  | `main`                           | The branch to work on. |
| `ssh_key_file`            | `string`  | `/home/yac/.ssh/id_rsa`          | Path to the private key file (SSH URLs only). |
| `ssh_known_hosts_file`    | `string`  | `/home/yac/.ssh/known_hosts`     | Path to the known hosts file (SSH URLs only). |
| `dirty_max_age`           | `integer` | `0`                              | Acceptable age (in minutes) of the last git fetch where a dirty read will not update the data again. |

## Key `connection` (plugin: `git_redis`)

`git_redis` accepts every `git_direct` key above (it delegates writes
and the on-pod working tree to `git_direct`) plus the Redis-specific
keys below.

| Key                | Type      | Default              | Description |
|:-------------------|:----------|:---------------------|:------------|
| `redis_url`        | `string`  | `""` (**required**)  | URL of the Redis instance, e.g. `redis://host:6379/0`. Must be a value accepted by `redis.asyncio.from_url`. |
| `max_age_seconds`  | `integer` | `300`                | How long a snapshot may serve reads before a refresh pull is triggered. Stale-but-ready snapshots are served while the refresh runs. |
| `grace_seconds`    | `integer` | `60`                 | How long old snapshot keys are kept after a swap so in-flight readers can finish. |
| `pull_lock_ttl`    | `integer` | `120`                | TTL of the cross-pod stampede mutex. Must comfortably exceed the worst-case git pull time. |


{: .note}
`max_age_seconds` is the **staleness budget** for external commits
(anything pushed to git outside YAC). YAC-initiated writes always
refresh the snapshot immediately on writer-scope exit.

{: .warning}
The Redis key namespace is global (`latest`, `synced`, `pull_lock`,
`ready:{hash}`, `paths:{hash}`, `data:{hash}:{path}`). Use a
dedicated Redis instance — or at minimum a dedicated logical
database (`/0`, `/1`, ...) — per YAC deployment.

## Key `details` (plugins: `git_direct`, `git_redis`)

A mapping from entity-type name to a [Jinja2](../j2.md) template that
resolves to the YAML file path **inside the repository** for an entity of
that type.

Each template:

  - **must** contain the variable `name` (used to find/list/create the
    entity's YAML file);
  - **must not** contain a literal `*` (it is reserved as a glob marker
    when listing entities);
  - is rendered with the entity's name to obtain the file path on read,
    write, copy, link, rename and delete.

The path is interpreted relative to the repository root (no leading `/`).
Path components that escape the repository (e.g. via `..`) are rejected
at runtime.

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
