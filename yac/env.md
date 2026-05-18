---
parent: YAC
nav_order: 2
---

# Environment Variables

The base configuration of YAC is done via environment variables. They are
defined in `app/config.py` (`Settings`). Mostly, the rest of the configuration
is done through the [specs file](specs/index.md).

All variables are prefixed with `YAC_`. For nested values, the delimiter
`__` (double underscore) is used (e.g. `YAC_REPO__URL` for `repo.url`).

## General

| Variable                          | Type                                                | Default                                            | Description |
|:----------------------------------|:----------------------------------------------------|:---------------------------------------------------|:------------|
| `YAC_ROOT_PATH`                   | `string`                                            | `/`                                                | Root path under which the API is served (useful behind a reverse proxy). |
| `YAC_CORS_ORIGINS`                | `string` (comma separated)                          | `https://localhost`                                | List of allowed CORS origins. |
| `YAC_LOG_LEVEL`                   | `critical\|error\|warning\|info\|debug`             | `info`                                             | Log level. |
| `YAC_FORMAT_PLUGIN`               | `string`                                            | `plain`                                            | Name of the [format](plugins.md) plugin to use for log formatting (e.g. `plain` or `json_lines`). |
| `YAC_DEBUG_MODE`                  | `boolean`                                           | `false`                                            | **DANGEROUS**: Leaks sensitive data (incl. secrets) to the user; only use in development environments. |
| `YAC_SPECS`                       | `string`                                            | `./yac.yml`                                        | Path to the [specs file](specs/index.md). A leading `.` means the path is relative to the repository root (support depends on the `repo_plugin`). Otherwise, an absolute path on the container filesystem. |
| `YAC_ENV__*`                      | `string`                                            | -                                                  | Custom environment variables that become available in the specs-file as `env.*` (e.g. `YAC_ENV__MY_VAR` -> `env.my_var`). |

## Authentication (OpenID Connect)

| Variable                          | Type                                                | Default                                            | Description |
|:----------------------------------|:----------------------------------------------------|:---------------------------------------------------|:------------|
| `YAC_OIDC_URL`                    | `string`                                            | `https://localhost/.well-known/openid-configuration` | URL of the OpenID Connect discovery document. |
| `YAC_OIDC_CLIENT_IDS`             | `string` (comma separated)                          | `''`                                               | List of accepted `client_id`s (the first is used as default in the Swagger UI). |
| `YAC_OIDC_JWT_NAME`               | `string` (format-string)                            | `{name}`                                           | Format-string to extract the user's `name` from the JWT ID-Token. |
| `YAC_OIDC_JWT_FULL_NAME`          | `string` (format-string)                            | `{givenName} {surname}`                            | Format-string to extract the user's `full_name`. |
| `YAC_OIDC_JWT_FULL_NAME_FALLBACK` | `string` (format-string)                            | `{name}`                                           | Fallback format-string for `full_name` if `YAC_OIDC_JWT_FULL_NAME` cannot be rendered. |
| `YAC_OIDC_JWT_EMAIL`              | `string` (format-string)                            | `{mail}`                                           | Format-string to extract the user's `email`. |
| `YAC_OIDC_JWT_EMAIL_FALLBACK`     | `string` (format-string)                            | `{name}@localhost`                                 | Fallback format-string for `email` if `YAC_OIDC_JWT_EMAIL` cannot be rendered. |

## Repository

| Variable                          | Type                                                | Default                                            | Description |
|:----------------------------------|:----------------------------------------------------|:---------------------------------------------------|:------------|
| `YAC_REPO_PLUGIN`                 | `string`                                            | `git_direct`                                       | Name of the [repo](plugins.md) plugin to use (e.g. `git_direct` or `git_redis`). |
| `YAC_REPO__*`                     | `string`                                            | -                                                  | `repo_plugin`-specific configuration (see below). |

### Plugin `git_direct`

When using this plugin, every (worker) process will have its own repo copy in
`/repo/{pid}`. To optimize, mount a tmpfs at `/repo`, so all the data is
always in memory.

| Variable                          | Type        | Default                  | Description |
|:----------------------------------|:------------|:-------------------------|:------------|
| `YAC_REPO__URL`                   | `string`    | `''` (**required**)      | The HTTPS or SSH URL to the git repo. |
| `YAC_REPO__BRANCH`                | `string`    | `main`                   | The branch to work on. |
| `YAC_REPO__SSH_KEY_FILE`          | `string`    | `/home/yac/.ssh/id_rsa`      | Path to the private key file. |
| `YAC_REPO__SSH_KNOWN_HOSTS_FILE`  | `string`    | `/home/yac/.ssh/known_hosts` | Path to the known hosts file. |
| `YAC_REPO__DIRTY_MAX_AGE`         | `integer`   | `0`                      | Acceptable age (in minutes) of the last git fetch where a dirty read will not update the data again. |
