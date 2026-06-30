---
parent: File
nav_order: 4
has_children: true
---

# Section `types`

The `types` section is the heart of a YAC specification: it is a **list** of
entity types, and each type turns one kind of YAML file in your data
repository into a full REST API (and, through [VAYS](../../../../vays/index.md), a web UI)
for listing, creating, changing and deleting those files.

A *type* bundles everything YAC needs to manage that kind of file:

- how entities are **named** (`name_pattern`, `name_generated`,
  `name_generator`, ...),
- which **operations** are available (`create`, `edit`, `delete`),
- what is shown in the **UI** (`title`, `description`, `options`,
  `favorites`),
- optional **limits**, **logs** and **actions**.

The data **schema** itself (the fields of each entity) is *not* defined here —
it lives in the top-level [`schema`](../schema.md) section, which applies to the
whole instance.

{: .note}
Only manage *similar* types in one YAC instance. All types share the same
[`schema`](../schema.md), [`roles`](../roles.md), [`repo`](../repo.md) and
[`auth`](../auth.md). If your file kinds are structurally very different, run a
second YAC instance instead of forcing them into one.

Like everywhere in the specs, every string value is a
[j2-string](../../j2.md) and is rendered per request, so most fields can depend
on the caller, the request and the existing data. Which variables are
available in which field is listed in the
[templating variable matrix](../../j2.md#variables).

## Field overview

| Field | Type | Default | Description |
|:------|:-----|:--------|:------------|
| [`name`](#name-and-title) | string | *mandatory* | Identifier of the type, used in URLs (`/entity/{name}`). |
| [`title`](#name-and-title) | string | *mandatory* | Human-readable name shown in the UI. |
| [`description`](#description) | string | `''` | Markdown description shown in the UI. |
| [`name_pattern`](#entity-names) | regex | `^[a-zA-Z0-9_\-\.]{1,200}$` | Allowed entity names. |
| [`name_example`](#entity-names) | string | `''` | Example name shown as a placeholder in the UI. |
| [`name_generated`](#generated-names) | enum | `never` | Whether the server generates the name (`never` / `optional` / `enforced`). |
| [`name_generator`](#generated-names) | j2-string | `uuid()` | Expression that produces the name when generated. |
| [`create`](#enabling-operations) | bool | `true` | Allow creating entities of this type. |
| [`edit`](#enabling-operations) | bool | `true` | Allow editing entities of this type. |
| [`delete`](#enabling-operations) | bool | `true` | Allow deleting entities of this type. |
| [`options`](#options) | list | `[]` | Extra values shown per entity in the list view. |
| [`favorites`](#favorites) | list | `[]` | Prominent operation/action buttons (and their order) in the UI. |
| [`limits`](limits.md) | list | `[]` | Caps on the number or summed quota of entities. |
| [`logs`](logs.md) | list | `[]` | External log/status sources surfaced per entity. |
| [`actions`](actions.md) | list | `[]` | Custom operations that call out to HTTP or shell. |

{: .note}
The more involved fields — [`limits`](limits.md), [`logs`](logs.md) and
[`actions`](actions.md) — each have their own page (see the child pages of
this section).

A minimal type only needs `name` and `title`:

```yaml
types:
  - name: host
    title: Hosts
  - name: network
    title: Networks
```

## Name and title

`name` is the stable identifier of the type. It must match
`^[a-zA-Z0-9_\-\.]{1,200}$` and appears in every endpoint of the type
(`/entity/{name}`, `/entity/{name}/{entity}`, ...). `title` is the
human-readable label shown in the UI.

## Description

`description` is a Markdown string shown in the UI to explain the type to
users. It is purely informational.

```yaml
description: |
  A **host** is a single physical or virtual machine.
  See the [handbook](https://example.com/hosts) for details.
```

## Entity names

`name_pattern` is a regular expression every entity name of this type must
match. Keep it as strict as your storage allows — names usually become file
names in the repository.

`name_example` is shown as a placeholder/example in the UI to hint at the
expected format. It has no effect on validation.

```yaml
name_pattern: '^[a-z][a-z0-9-]{2,30}$'
name_example: web-server-01
```

### Generated names

By default (`name_generated: never`) the client must supply a valid name on
create. You can instead let YAC generate the name:

| `name_generated` | Behaviour on create |
|:-----------------|:--------------------|
| `never`          | The request **must** contain a valid name. |
| `optional`       | The request **may** contain a name; if it is `null`, one is generated. |
| `enforced`       | The request **must not** contain a name; it is always generated. |

When a name is generated, it is produced by the `name_generator`
[j2-string](../../j2.md) and must still match `name_pattern` (and be unique).
The default generator is `uuid()`. Inside the generator you additionally have
access to `old.list` (the existing names of the type) and `new.data` (the data
being created) — see the [variable matrix](../../j2.md#variables).

{% raw %}
```yaml
# Derive a slug from a data field:
name_generated: enforced
name_generator: "new.data.title | lower | regex_replace(' ', '-')"

# Or count up to the next free number, capped at 50 entities:
name_generated: enforced
name_generator: "'host-' ~ re_next_int('^host-(\\d+)$', limit=50)"
```
{% endraw %}

{: .note}
For a *maximum number of entities* that also works with client-supplied names
and data-based quotas, prefer the dedicated [`limits`](limits.md) field over
the `limit` argument of `re_next_int` (which only applies while generating a
name).

## Enabling operations

`create`, `edit` and `delete` are coarse on/off switches for the
corresponding operations of the **whole type**. Turning one off removes the
operation for everyone, regardless of permissions.

{: .important}
These switches are *not* a substitute for access control. Even when an
operation is enabled, the caller still needs the matching permission from the
[`roles`](../roles.md) section (see [Permissions](../../perms.md)). Use the switches
to disable an operation entirely (e.g. a read-only, externally-maintained
type), and use roles to decide *who* may perform the enabled ones.

```yaml
# A read-only type: entities are listed and viewed, never written through YAC.
create: false
edit: false
delete: false
```

## options

`options` are extra values attached to every entity when **listing** them
(`GET /entity/{type}`), so the UI can show meaningful columns without fetching
each entity in full. Each option reads one value from the entity's data:

| Field | Description |
|:------|:------------|
| `name` | *mandatory*; the data key to read. |
| `title` | Column title in the UI. |
| `default` | Value to show when the entity has not set this key. |
| `aliases` | Map of raw value → Markdown label, to prettify displayed values. |

```yaml
options:
  - name: os
    title: Operating System
    default: unknown
    aliases:
      deb12: Debian 12
      u2404: Ubuntu 24.04
```

## favorites

`favorites` controls which operations and actions get prominent buttons in the
UI, and in which order. Each entry is either a **standard operation** or a
**custom action** (`action: true`):

- Standard operations: `create_copy`, `create_link`, `edit`, `delete`.
  (`create_new` cannot be a favorite — it is a type-level, not entity-level,
  operation.)
- Custom actions: any `name` defined in this type's [`actions`](#actions).

```yaml
favorites:
  - name: edit            # standard operation
  - name: delete          # standard operation
  - name: reinstall       # a custom action defined below
    action: true
```

## limits

`limits` caps **how many** entities of this type may exist within a group, or
**how much** of a summed quota a group may use — for example "at most 5 VMs
per owner" or "at most 100 GB of disk per owner". Because limits have several
interacting expressions and their own performance and enforcement semantics,
they are documented on a dedicated page:

See the dedicated page: [Field `limits`](limits.md)

```yaml
limits:
  - title: VMs per owner
    scope: "old.data.owner == new.data.owner"
    max: "5"
```

## logs

`logs` surface external log or status information for an entity (e.g. an
install log or a health check) so it can be shown in the UI, read from a
pluggable backend (`elastic` or `file`).

See the dedicated page: [Field `logs`](logs.md)

## actions

`actions` are custom operations beyond plain CRUD: they call out to an HTTP
endpoint or run a shell command, either on demand or **hooked** to run
automatically around create/edit/delete.

See the dedicated page: [Field `actions`](actions.md)

## Full example

A complete `animal` type exercising most fields:

{% raw %}
```yaml
types:
  - name: animal
    title: Animals
    description: A single animal tracked by the **zoo** system.
    name_pattern: '^[a-z][a-z0-9-]{1,30}$'
    name_example: lion-simba

    create: true
    edit: true
    delete: true

    limits:
      - title: Animals per keeper
        scope: "old.data.keeper == new.data.keeper"
        max: "10"

    options:
      - name: color
        title: Color
        default: grey
        aliases:
          grey: Grey
          brown: Brown
          red: Red

    favorites:
      - name: flush
        action: true
      - name: delete

    logs:
      - name: camera-trap
        title: Camera Trap
        progress: true
        plugin: elastic
        details:
          url: "https://user:{{ env.elastic_pass }}@elastic.example.com:9200/camera-trap"
          query: 'any where animal.name == "{{ name }}"'
          # `log` here is the elastic hit's `_source` dict.
          message: "![image]({{ log.animal.foto_url }})"
          progress: "{{ log.animal.count * 100 / log.animal.total }}"
      - name: camera-status
        title: Camera Status
        problem: true
        plugin: file
        details:
          path: "/logs/camera/{{ name }}.log"
          # `line_format` is a regex; capture groups become the `log` tuple.
          line_format: '^\[([^\]]+)\] (.*)$'
          time: "{{ log[0] }}"
          message: "{{ log[1] }}"
          problem: "{{ log[1] is regex_match('.*failed.*') }}"

    actions:
      - name: flush
        title: Flush Fotos
        description: Flush all photos from the [Gallery]({{ env.photo_gallery_url }})
        dangerous: true
        perms: [act, adm]
        force: false
        hooks:
          - arbitrary
          - delete:after
        plugin: http
        details:
          method: POST
          url: "https://photos.example.com/api/{{ name }}/?secret={{ env.photo_secret }}"
          body: '{"action": "flush", "animal": "{{ name }}"}'
          headers:
            Content-Type: application/json
          error: [409]
      - name: hello
        title: Say Hello
        hooks:
          - create:before
        plugin: shell
        details:
          command: 'echo "Hello {{ user.full_name }}"'
```
{% endraw %}
