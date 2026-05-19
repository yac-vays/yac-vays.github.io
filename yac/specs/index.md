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
| [`request`](file/request.md)| no       | Custom HTTP headers exposed to the templating engine. |
| [`types`](file/types.md)    | yes      | Entity types managed by this YAC instance, including their actions and logs. |
| [`repo`](file/repo.md)      | no       | Repository plugin selection, connection config and per-type path templates. |
| [`auth`](file/auth.md)      | no       | OpenID Connect authentication and CORS origins. |
| [`roles`](file/roles.md)    | no       | Run-time permission rules per entity. |
| [`sets`](file/sets.md)      | no       | Named entity subsets referenced from `roles`. |
| [`context`](file/context.md)| no       | Static variables made available to the templating engine. |
| [`schema`](file/schema.md)  | yes      | JSON-Schema (Draft-7 + extensions) used for validation and form generation. |

The values in any of these sections can use [Jinja2 templating](j2.md) (with
the exception of the `context` section itself).

For a quick visual overview, see [Examples](examples.md). For the
permission model used by `roles`, `sets` and `schema`, see
[Permissions](perms.md).

## Authoritative source

This documentation is the authoritative reference for the specs file format.
Code-level docstrings and field descriptions in YAC's source (e.g.
`app/model/`) feed the OpenAPI/Swagger UI and may be terser; in case of
divergence, this documentation wins.
