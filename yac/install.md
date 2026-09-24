---
parent: YAC
nav_order: 1
---

# Installation

YAC is distributed as a container image. Configuration is provided via
[environment variables](env.md) and a [specs file](specs/index.md) mounted
into the container (default path: `/yac.yml`). The specs file is static —
changing it requires a pod/container restart.

## Prerequisites

  - A Git repository (with a write-access token/key for YAC) for storing the
    managed YAML files.
  - An OpenID Connect (OIDC) provider with at least one client configuration for
    YAC (and VAYS) to handle authentication.
  - A [specs file](specs/index.md) (`yac.yml`) specifying and configuring everything.

## Plain Docker

A minimal example to start YAC locally:

```sh
sudo docker run --rm --name yac -p 127.0.0.1:8080:8080 \
    -v /path/to/yac.yml:/yac.yml:ro \
    --tmpfs /repo \
    registry.inf.ethz.ch/public-isg/yac-vays/yac:latest
```

The API and Swagger UI documentation are then available at
[http://localhost:8080/](http://localhost:8080/).

Hints:

  - Mount a tmpfs at `/repo` so the working copies (one per worker) are kept
    in memory (recommended for the `git_direct` repo plugin).
  - For SSH access to the repo, mount your private key and `known_hosts`
    and configure their paths in the specs file's `repo.connection`
    (`ssh_key_file` / `ssh_known_hosts_file`).
  - To install [plugins](plugins.md), bind-mount them into
    `/code/app/plugin/{type}/{name}.py` inside the container.
  - To put the specs file at a non-default location, mount it there and
    set `YAC_SPECS` to its absolute path.

## Helm

A Helm chart is provided in the `helm/` directory of the repository and
on registry.inf.ethz.ch.

See [values.yaml](https://github.com/yac-vays/yac/blob/main/helm/values.yaml)
for possible variables, default values and details.

The chart accepts an optional `specs:` value: when set to a YAML string,
it is written to a ConfigMap and mounted into the pod at `/yac.yml` (the
default specs path). Leave it empty to mount the specs file from outside
the chart via `extraVolumes` / `extraVolumeMounts`, or to bake it into a
custom container image.

{% raw %}
For secrets the specs file should reference via `{{ env.* }}` (tokens,
LDAP passwords, ...), use either `customEnv` (the chart creates a
managed `Secret`) or `extraEnvSecret` (the chart loads an existing,
externally-managed `Secret` via `envFrom`). The chart-managed `Secret`
is only created when at least one of `env` / `customEnv` is non-empty.
{% endraw %}

```sh
helm show chart oci://registry.inf.ethz.ch/public-isg/yac-vays/yac/charts/yac
helm install yac oci://registry.inf.ethz.ch/public-isg/yac-vays/yac/charts --version <VERSION> -f values.yaml
```

### Redis (optional, for the `git_redis` repo plugin)

The chart can optionally deploy or wire up a Redis instance used by the
[`git_redis`](specs/file/repo.md#connection-git_redis) repo plugin.
When enabled, the chart sets `YAC_ENV__REDIS_URL` on the YAC pod so the
specs file can reference it as `env.redis_url`.

```yaml
redis:
  enabled: true
  mode: single        # or: external
  # url: "redis://my-existing-redis:6379/0"   # required when mode=external
```

Modes:

  - **`single`** (default): the chart deploys a single-pod Redis
    `StatefulSet` + `Service` next to YAC. By default this uses an
    `emptyDir` (cache semantics — if Redis dies, the next reader
    rebuilds the snapshot from git). Persistence is opt-in via
    `redis.single.persistence.enabled: true`. Zero extra dependencies.
  - **`external`**: the chart does not deploy Redis; you supply
    `redis.url`. Use this to point YAC at a Redis HA cluster you
    deploy separately.

{: .important}
Enabling the `redis:` block in `values.yaml` is only half the wiring —
your specs file must also select `git_redis` and reference the env var.
The full Redis-side tuning (`max_age_seconds`, `grace_seconds`,
`pull_lock_ttl`) is set directly in the specs file; see
[Section `repo` → `connection` (`git_redis`)](specs/file/repo.md#connection-git_redis).

{% raw %}
```yaml
# In your specs file (mounted via chart `specs:` value or extraVolumes)
repo:
  plugin: git_redis
  connection:
    url: "https://yac:{{ env.GIT_TOKEN }}@git.example.com/my/repo.git"
    branch: main
    redis_url: "{{ env.redis_url }}"
    max_age_seconds: 300
  details:
    animal: "animals/{{ name }}.yml"
```
{% endraw %}

## Health Checks and Monitoring

YAC has two unauthenticated endpoints for this, with different purposes:

  - `GET /health` answers `204` as soon as the process serves requests. It
    does no repository or specs work and is what the Helm chart's
    liveness, readiness and startup probes use. A pod is never restarted
    because of the git server: that is nothing a restart could fix (see
    [Remote outages](specs/file/repo.md#remote-outages)).
  - `GET /status` is the diagnostic endpoint for monitoring. It parses the
    specs and accesses the repository and answers:

    ```json
    {
      "hash": "<git hash of the data>",
      "repo": {
        "available": true,
        "stale": false,
        "synced": "2026-09-24T10:00:00Z",
        "error": null
      }
    }
    ```

    | Situation | Status | `hash` | `repo.available` | `repo.stale` |
    |:----------|:-------|:-------|:-----------------|:-------------|
    | Everything fine | `200` | set | `true` | `false` |
    | Remote down, reads served from the last known state | `200` | set | `false` | `true` |
    | Remote down and no data at all (never cloned) | `503` | `null` | `false` | `false` |

    `repo.synced` is the time of the last successful sync with the remote,
    `repo.error` the user-facing reason while the remote is unavailable
    (it distinguishes maintenance from an unreachable remote; the detailed
    git output is only in the pod log). Do **not** use `/status` as a
    liveness probe.

## Versioning / Container Tags

The container images are available with the following tag schema:

  - *latest*: The latest stable release
  - *v1*, *v2*, ...: A specific major release (stable API)
  - *v1.0*, *v2.1*, ...: A specific minor version
  - *testing*: The latest testing release
  - *v2rc*, *v7rc*, ...: A specific major testing release
  - *v2.0rc*, *v3.11rc*, ...: A specific minor testing release

For example:

```sh
docker pull registry.inf.ethz.ch/public-isg/yac-vays/yac:v1
```
