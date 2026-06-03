---
parent: Section types
nav_order: 1
---

# Field `limits`

`limits` is a per-[type](types.md) field that caps **how many entities** of a
type may exist within a group, or **how much of a summed quota** a group may
use. Limits are enforced on every `create` and `change` operation (a `delete`
can never exceed a cap) and surface as a normal validation error.

A limit is the combination of three Jinja2 expressions:

- **`scope`** — decides which existing entities belong to the same group as
  the entity being created/changed (think of it as a [set](sets.md)).
- an **aggregate** — either `count` (number of entities) or `sum` of a
  per-entity **`value`**.
- **`max`** — the cap, which may itself depend on the user or context (e.g. a
  per-user or per-plan quota).

All three are rendered with the same variables as [`roles`](roles.md) /
[`sets`](sets.md): `old`, `new`, `name`, `user`, `context`, `env`, `request`.
While aggregating, `old` is the **existing entity being scanned** and `new` is
the **incoming entity** being created or changed. The reported usage always
includes the incoming entity itself, so the UI can show e.g. `3/5 used`.

## Defaults

{% raw %}
```yaml
limits:
  - title:           # mandatory; shown in the error message and the UI
    scope: 'true'    # j2 test: does the scanned entity (old) count? (default: all)
    aggregate: count # or: sum
    value: '1'       # j2 number: a scanned entity's contribution (only for sum)
    max:             # mandatory; j2 number: the cap (may depend on user/context)
    on: [create, change]  # operations this limit is enforced on
```
{% endraw %}

## Example: a maximum number of entities per owner

Allow each owner at most five `vm` entities. `scope` groups entities by their
`owner` field; `aggregate: count` is the default.

{% raw %}
```yaml
types:
  - name: vm
    title: Virtual Machines
    limits:
      - title: VMs per owner
        scope: "old.data.owner == new.data.owner"
        max: "5"
```
{% endraw %}

To cap the total number of entities globally (regardless of owner), drop the
`scope` so every entity counts:

{% raw %}
```yaml
    limits:
      - title: Total number of VMs
        max: "100"
```
{% endraw %}

To make the cap depend on who is acting — for example via an LDAP/group lookup
exposed in `context` — let `max` reference `user`:

{% raw %}
```yaml
    limits:
      - title: VMs per owner
        scope: "old.data.owner == new.data.owner"
        max: "{{ context.vm_quota(user.name) }}"
```
{% endraw %}

## Example: a maximum summed quota per owner

Sum a numeric field (`disk_gb`) over all of an owner's entities and cap the
total. `value` is the contribution of each scanned entity; the incoming
entity's own value is included automatically.

{% raw %}
```yaml
types:
  - name: vm
    title: Virtual Machines
    limits:
      - title: Disk quota per owner (GB)
        scope: "old.data.owner == new.data.owner"
        aggregate: sum
        value: "old.data.disk_gb | default(0)"
        max: "{{ context.disk_quota_gb }}"
```
{% endraw %}

Grouping is not limited to a `data` field — use any request variable. For a
per-OU cap, for instance:

{% raw %}
```yaml
    limits:
      - title: VMs per organisational unit
        scope: "old.data.ou == user.token.ou"
        max: "20"
```
{% endraw %}

## Semantics

- The candidate set is **every existing entity of the type, minus the one
  being replaced on a `change`, plus the incoming entity.** This makes
  create, rename, owner-change and quota-increase all behave correctly: an
  edit that keeps an entity in its group does not falsely trip the limit,
  while moving it into an already-full group does.
- `used` in the result and the error message already counts the incoming
  entity, so `max: "5"` permits five entities (the sixth `create` fails with
  `Limit "…" reached: 6/5 entities.`).
- Limits run **after** permission and conflict checks, so a forbidden or
  conflicting operation reports its own, more specific error first.

{: .note}
Limits are also evaluated by the `POST /validate` endpoint, so VAYS can show
the live usage of every applicable limit (`schemas`/`request`/**`usages`** in
the response) while the user fills in the form — including a friendly
`3/5 used` indicator and an error before the operation is even submitted.

## Performance

Counting and summing scans the entities of the type. To keep this cheap:

{: .note}
Prefer a `scope` (and, for `sum`, a `value`) that references **`old.name`**,
`user` or `context` rather than `old.data`. When no expression of a limit
touches `old.data`, YAC counts straight from the name list **without loading
or parsing any entity YAML**. Limits that do reference `old.data` load each
entity's data with the same bounded concurrency and content-keyed cache used
by the entity-list endpoint.

{: .warning}
The count/sum is measured when the request is validated and enforced just
before the write is committed — the same small window that already exists for
name-uniqueness checks. Two writes racing at exactly the cap could both pass.
For most workloads this is acceptable; if you need a hard guarantee, keep the
cap enforcement paired with a unique `name_pattern`/`name_generator` so the
commit itself conflicts.
