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

  - A Git repository (writeable by YAC) for storing the managed YAML files.
    The git URL, branch, SSH key path, etc. are configured in the [specs
    file's `repo` section](specs/file/repo.md).
  - An OpenID Connect (OIDC) provider for authentication.
  - A specs file describing the repository, entity types, roles and
    schemas (see [Specification](specs/index.md)).

## Plain Docker

A minimal example (OpenID Connect URL, accepted client IDs, the data
repo URL, ... all live in `yac.yml`):

```sh
docker run --rm --name yac -p 8080:80 \
    -v /path/to/yac.yml:/yac.yml:ro \
    --tmpfs /repo \
    yacvays/yac:latest
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
