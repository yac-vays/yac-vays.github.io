---
parent: VAYS
nav_order: 1
---

# Installation

VAYS is shipped as a container image that serves the built static bundle
through `nginx` (using the unprivileged image, listening on port `8080`).
Configuration happens entirely through a single file, `config.json`,
which is loaded by the browser at start-up — see
[Configuration](config.md) for its reference.

## Prerequisites

  - At least one running [YAC](../yac/index.md) backend reachable from the
    user's browser.
  - An OpenID Connect (OIDC) provider that both VAYS and the YAC backends
    trust.
  - A `config.json` file (see [Configuration](config.md) for the schema
    and an example).

## Plain Docker

```sh
docker run --rm --name vays -p 8080:8080 \
    --mount type=bind,source="$(pwd)/config.json",target=/usr/share/nginx/html/config.json,readonly \
    yacvays/vays:latest
```

VAYS is then reachable at [http://localhost:8080/](http://localhost:8080/).

Notes:

  - The redirect URI configured at the OIDC provider must match the URL at
    which the user opens VAYS (typically `https://<your-host>/oauth2-redirect`).
    The OIDC provider will likely require HTTPS, so a development setup
    behind a reverse proxy or a self-signed cert is the easiest option.
  - The VAYS image only serves static assets; CORS is configured on the
    YAC side via [`YAC_CORS_ORIGINS`](../yac/env.md).

## Helm

A Helm chart is provided in the `helm/` directory of the repository and
on registry.inf.ethz.ch.

See [values.yaml](https://github.com/yac-vays/vays/blob/main/helm/values.yaml)
for possible variables, default values and details.

```sh
helm show chart oci://registry.inf.ethz.ch/public-isg/yac-vays/vays/charts/vays
helm install vays oci://registry.inf.ethz.ch/public-isg/yac-vays/vays/charts --version <VERSION> -f values.yaml
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
docker pull registry.inf.ethz.ch/public-isg/yac-vays/vays:v1
```
