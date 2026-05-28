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
  - An OpenID Connect (OIDC) provider with a client configuration for this
    VAYS and all attached YAC instances to handle authentication.
  - A [config file](config.md) (`config.json`).

## Plain Docker

A minimal example to start VAYS locally:

```sh
sudo docker run --rm --name vays -p 127.0.0.1:8080:8080 \
    -v /path/to/config.json:/usr/share/nginx/html/config.json:ro \
    registry.inf.ethz.ch/public-isg/yac-vays/vays:latest
```

VAYS is then reachable at [http://localhost:8080/](http://localhost:8080/).

Hints:

  - The redirect URI configured at the OIDC provider must match the URL at
    which the user opens VAYS (typically `https://<your-host>/oauth2-redirect`).
    The OIDC provider will likely require HTTPS, so a development setup
    behind a reverse proxy or a self-signed cert is the easiest option.
  - The VAYS image only serves static assets; CORS is configured on the
    YAC side via [`auth.cors.origins`](../yac/specs/file/auth.md) in
    the specs file.

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
