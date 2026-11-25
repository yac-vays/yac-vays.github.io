---
parent: MCP
nav_order: 1
---

# Installation

The MCP server is shipped as a container image (listening on port `8080`).
It is stateless: several replicas only share the envelope keys (see below).

## Prerequisites

  - At least one running [YAC](../yac/index.md) backend reachable from the
    MCP server.
  - A **confidential** client at the OIDC provider (the same provider as YAC
    and VAYS) with the redirect URI `https://<mcp-host>/callback` and the
    scopes for the claims the YAC specs use. The provider must return an
    id-token on refresh, else users have to log in again when it expires.
  - The client ID in [`auth.oidc.client_ids`](../yac/specs/file/auth.md) of
    every YAC backend (the user's id-tokens are passed on to YAC).
  - A config file `config.json` with the backends: the same format as the
    [VAYS config](../vays/config.md), of which only `backends[]` (`name`,
    `title`, `url`) and `yamlComment` are used.

## Configuration

Environment variables (all with the prefix `YAC_MCP_`):

| Variable | Default | Description |
|:---------|:--------|:------------|
| `YAC_MCP_PUBLIC_URL` | `http://localhost:8080` | External URL of the server (OAuth issuer; the MCP endpoint is `{url}/mcp`). |
| `YAC_MCP_CONFIG` | `/config.json` | Path of the config file. |
| `YAC_MCP_OIDC_URL` | | Discovery URL of the OIDC provider. |
| `YAC_MCP_OIDC_CLIENT_ID` / `_SECRET` | | The confidential client. |
| `YAC_MCP_OIDC_SCOPE` | `openid email` | Scopes requested at the login. |
| `YAC_MCP_KEYS` | | JSON list of envelope keys, e.g. `["kid:base64url"]`; generate one with `python -m app.lib.envelope` in the container. The first key encrypts, all decrypt (rotation). |
| `YAC_MCP_READ_ONLY` | `true` | Disables the write tools. |
| `YAC_MCP_REVEAL_GENERATED_SECRETS` | `false` | Return generated secrets to the agent. |
| `YAC_MCP_PLUGIN_DIRS` | `[]` | Additional plugin roots (JSON list). |
| `YAC_MCP_LOG_LEVEL` | `info` | Log level. |

See [config.py](https://github.com/yac-vays/mcp/blob/main/app/config.py) for
all options.

## Plain Docker

```sh
sudo docker run --rm --name mcp -p 127.0.0.1:8080:8080 \
    -v /path/to/config.json:/config.json:ro \
    -e YAC_MCP_PUBLIC_URL=https://mcp.example.com \
    -e YAC_MCP_OIDC_URL=https://idp.example.com/.well-known/openid-configuration \
    -e YAC_MCP_OIDC_CLIENT_ID=yac-mcp -e YAC_MCP_OIDC_CLIENT_SECRET=... \
    -e YAC_MCP_KEYS='["..."]' \
    registry.inf.ethz.ch/public-isg/yac-vays/mcp:latest
```

TLS is expected to be terminated in front of the container, at the root of
its host (no path prefix).

## Helm

A Helm chart is provided in the `helm/` directory of the repository and
on registry.inf.ethz.ch.

See [values.yaml](https://github.com/yac-vays/mcp/blob/main/helm/values.yaml)
for possible variables, default values and details.

```sh
helm show chart oci://registry.inf.ethz.ch/public-isg/yac-vays/mcp/charts/mcp
helm install mcp oci://registry.inf.ethz.ch/public-isg/yac-vays/mcp/charts/mcp --version <VERSION> -f values.yaml
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
docker pull registry.inf.ethz.ch/public-isg/yac-vays/mcp:v1
```
