---
parent: File
nav_order: 3
---

# Section `repo`

The `repo` section configures the repository plugin (selected via the
[`YAC_REPO_PLUGIN`](../../env.md) env var).  Plugin-specific options live
under `repo.details`.

The shape of `repo.details` is plugin-specific; the rest of this page
documents the format used by the bundled `git_direct` and `git_redis`
plugins (which both read the same details).

## Per-type file paths (`git_direct` / `git_redis`)

For these plugins, `repo.details` is a mapping from entity type name to a
[Jinja2](../j2.md) template that resolves to the YAML file path **inside
the repository** for an entity of that type.

Each template:

  - **must** contain the variable `name` (used to find/list/create the
    entity's YAML file);
  - **must not** contain a literal `*` (it is reserved as a glob marker
    when listing entities);
  - is rendered with the entity's name to obtain the file path on read,
    write, copy, link, rename and delete.

The path is interpreted relative to the repository root (no leading `/`).
Path components that escape the repository (e.g. via `..`) are rejected at
runtime.

## Example

{% raw %}
```yaml
repo:
  details:
    animal: "animals/{{ name }}.yml"
    cage:   "cages/{{ name }}/cage.yml"
```
{% endraw %}

With this configuration, an entity of type `animal` named `rex` is stored
at `animals/rex.yml`, and a `cage` named `c1` at `cages/c1/cage.yml`.

## Notes

  - Every entity type defined under [`types`](types.md) must have a
    corresponding entry in `repo.details`.
  - Other repository-related settings (URL, branch, SSH key, refresh
    interval, ...) are configured via [environment variables](../../env.md),
    not in this section.

{: .warning}
TODO All repository-related settings will be moved here as soon as the
following feature has been removed: having the specs-file inside the
repo.
