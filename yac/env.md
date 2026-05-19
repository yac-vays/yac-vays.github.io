---
parent: YAC
nav_order: 2
---

# Environment Variables

The base configuration of YAC is done via environment variables. They are
defined in `app/config.py` (`Settings`). Mostly, the rest of the configuration
is done through the [specs file](specs/index.md).

All variables are prefixed with `YAC_`. For nested values, the delimiter
`__` (double underscore) is used (e.g. `YAC_ENV__MY_VAR` for `env.my_var`).

## General

| Variable           | Type                                    | Default | Description |
|:-------------------|:----------------------------------------|:--------|:------------|
| `YAC_ROOT_PATH`    | `string`                                | `/`     | Root path under which the API is served (useful behind a reverse proxy). |
| `YAC_LOG_LEVEL`    | `critical\|error\|warning\|info\|debug` | `info`  | Log level. |
| `YAC_FORMAT_PLUGIN`| `string`                                | `plain` | Name of the [format](plugins.md) plugin to use for log formatting (e.g. `plain` or `json_lines`). |
| `YAC_DEBUG_MODE`   | `boolean`                               | `false` | **DANGEROUS**: Leaks sensitive data (incl. secrets) to the user; only use in development environments. |
| `YAC_SPECS`        | `string`                                | `/yac.yml` | Absolute filesystem path of the [specs file](specs/index.md) inside the container. The specs file is static — changing it requires a pod/container restart. |
| `YAC_ENV__*`       | `string`                                | -       | Custom environment variables that become available in the specs-file as `env.*` (e.g. `YAC_ENV__MY_VAR` -> `env.my_var`). Use this for secrets you want to keep out of the specs file (tokens, passwords, ...). |

## Repository

Repository plugin selection and connection details live in the
[specs file](specs/file/repo.md), not in environment variables.

## Authentication / CORS

OpenID Connect (URL, accepted client IDs, JWT claim format-strings) and
CORS origins live in the specs file under [`auth`](specs/file/auth.md),
not in environment variables.
