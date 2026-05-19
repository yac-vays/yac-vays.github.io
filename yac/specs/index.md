---
parent: YAC
nav_order: 3
---

# Specification

The **specs** file is the main configuration file for a YAC instance. It is
a YAML file mounted into the container (by default at `/yac.yml` — see
[`YAC_SPECS`](../env.md)). The specs file is **static**: it is loaded once
at process startup, so changes to it require a pod/container restart.

It describes everything that is specific to a particular YAC deployment:
which repository (plugin, URL, branch, ...) is used, which entity types
exist and where their YAML files live in the repository, which roles and
permissions apply, the (extended) JSON-Schema used for validation and
form generation, and any custom context, sets, request headers and
templating data.

## Sections

The specs file consists of the following top-level sections (each documented
on its own page):

| Section                     | Required | Purpose |
|:----------------------------|:---------|:--------|
| [`auth`](file/auth.md)      | no       | OpenID Connect authentication and CORS origins. |
| [`repo`](file/repo.md)      | no       | Repository plugin selection, connection config and per-type path templates. |
| [`request`](file/request.md)| no       | Custom HTTP headers exposed to the templating engine. |
| [`types`](file/types.md)    | yes      | Entity types managed by this YAC instance, including their actions and logs. |
| [`roles`](file/roles.md)    | no       | Run-time permission rules per entity. |
| [`sets`](file/sets.md)      | no       | Named entity subsets referenced from `roles`. |
| [`context`](file/context.md)| no       | Static variables made available to the templating engine. |
| [`schema`](file/schema.md)  | yes      | JSON-Schema (Draft-7 + extensions) used for validation and form generation. |

The values in any of these sections can use [Jinja2 templating](j2.md) (with
the exception of the `context` section itself).

For a quick visual overview, see [Examples](examples.md). For the
permission model used by `roles`, `sets` and `schema`, see
[Permissions](perms.md).

## Splitting the specs across files (`yac_include`)

A `yac_include` key can be placed inside any object (at any nesting level)
to merge in YAML from one or more sibling files. This is useful for
keeping large specs readable, sharing common context/schema fragments
between deployments, or generating fragments separately.

The value is either a single path or a list of paths, **relative to the
file containing the `yac_include` key**. All included paths must resolve
to locations under the directory of the main specs file (set via
[`YAC_SPECS`](../env.md)); paths pointing outside that root cause YAC to
fail at startup.

{: .note}
Includes are merged at process startup, **before** any
[Jinja2 rendering](j2.md) happens. The path value itself therefore
cannot contain Jinja2 expressions — it must be a literal string.
The *content* of an included file can use Jinja2 as usual: after the
merge, it is rendered together with the section it ends up in.

Merge rules:

  - Included files are processed recursively — an included file can
    itself use `yac_include`.
  - When the included content is an **object**, its keys are merged into
    the parent object. Keys that already exist in the parent are **kept**
    (the parent wins); the include only fills in missing keys.
  - When the included content is a **non-object** (e.g. a list or scalar),
    it replaces the parent object only if that parent is otherwise empty
    and exactly one path is listed. Otherwise the include is skipped with
    a warning.
  - When multiple paths are listed and several provide the same key, the
    first include in the list wins for that key.

Example — split the schema and a context fragment into separate files:

{% raw %}
```yaml
# yac.yml
version: 1
types:
  - name: animal
    title: Animal

context:
  yac_include: context.fragment.yml

schema:
  yac_include:
    - schema.common.yml
    - schema.animal.yml
```
{% endraw %}

The specs file is loaded once at process startup, so changes to included
files require a pod/container restart, exactly like changes to the main
specs file.

## Authoritative source

This documentation is the authoritative reference for the specs file format.
Code-level docstrings and field descriptions in YAC's source (e.g.
`app/model/`) feed the OpenAPI/Swagger UI and may be terser; in case of
divergence, this documentation wins.
