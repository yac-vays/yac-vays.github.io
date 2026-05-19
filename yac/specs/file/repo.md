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

## `plugin`

The repository plugin to use. Built-in plugins:

  - `git_direct` (default): every worker clones the repo into
    `/repo/{pid}` and reads/writes files directly. Recommended for most
    deployments. Mount a tmpfs at `/repo` so the working copies live in
    memory.
  - `git_redis`: experimental cache layer in front of `git_direct`.

## `connection` (`git_direct`)

| Key                       | Type      | Default                          | Description |
|:--------------------------|:----------|:---------------------------------|:------------|
| `url`                     | `string`  | `""` (**required**)              | HTTPS or SSH URL to the git repo. |
| `branch`                  | `string`  | `main`                           | The branch to work on. |
| `ssh_key_file`            | `string`  | `/home/yac/.ssh/id_rsa`          | Path to the private key file (SSH URLs only). |
| `ssh_known_hosts_file`    | `string`  | `/home/yac/.ssh/known_hosts`     | Path to the known hosts file (SSH URLs only). |
| `dirty_max_age`           | `integer` | `0`                              | Acceptable age (in minutes) of the last git fetch where a dirty read will not update the data again. |

## `details` (`git_direct` / `git_redis`)

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
    url: "https://yac:{{ env.GIT_TOKEN }}@git.example.com/my/repo.git"
    branch: main
  details:
    animal: "animals/{{ name }}.yml"
```
{% endraw %}

## Notes

  - Every entity type defined under [`types`](types.md) must have a
    corresponding entry in `repo.details`.
  - To keep secrets (e.g. a git URL with credentials) out of the
    specs file, mount the specs file from a Kubernetes `Secret` (using
    `extraVolumes` / `extraVolumeMounts`) instead of the chart's `specs:`
    value, or bake them into a custom container image.
