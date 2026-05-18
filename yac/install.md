---
parent: YAC
nav_order: 1
---

# Installation

YAC is distributed as a container image. Configuration is provided via
[environment variables](env.md) and a [specs file](specs/index.md) (a YAML
file located inside the data repository or mounted into the container).

## Prerequisites

  - A Git repository (writeable by YAC) for storing the managed YAML files.
  - An OpenID Connect (OIDC) provider for authentication.
  - A specs file describing the entity types, roles and schemas (see
    [Specification](specs/index.md)).

## Plain Docker

A minimal example:

```sh
docker run --rm --name yac -p 8080:80 \
    --env YAC_REPO__URL="https://user:pass@git.example.com/my/repo.git" \
    --env YAC_OIDC_URL="https://example.com/.well-known/openid-configuration" \
    --env YAC_OIDC_CLIENT_IDS="my-client-id" \
    --tmpfs /repo \
    yacvays/yac:latest
```

The API and Swagger UI documentation are then available at
[http://localhost:8080/](http://localhost:8080/).

Hints:

  - Mount a tmpfs at `/repo` so the working copies (one per worker) are kept
    in memory (recommended for the `git_direct` repo plugin).
  - For SSH access to the repo, mount your private key and `known_hosts` and
    set `YAC_REPO__SSH_KEY_FILE` / `YAC_REPO__SSH_KNOWN_HOSTS_FILE`
    accordingly.
  - To install [plugins](plugins.md), bind-mount them into
    `/code/app/plugin/{type}/{name}.py` inside the container.
  - To use a specs file outside the data repository, mount it into the
    container and set `YAC_SPECS` to its absolute path (e.g. `/yac.yml`).

## Helm

A Helm chart is provided in the `helm/` directory of the repository and
on registry.inf.ethz.ch.

See [values.yaml](https://github.com/yac-vays/yac/blob/main/helm/values.yaml)
for possible variables, default values and details.

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
