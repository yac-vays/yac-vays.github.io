---
parent: Specification
nav_order: 2
---

# Permissions

YAC enforces permissions on two independent levels:

1. **File-level**: May this user perform this operation (read, create, edit,
   delete, run an action) on this entity at all? This is computed at run-time
   from the [roles](file/roles.md) and [sets](file/sets.md) sections of the
   specs.
2. **Property-level**: Which parts of the entity data may this user *write*?
   This is defined with the [`yac_perms`](file/schema.md#keyword-yac_perms)
   keyword inside the schema.

A write operation must pass **both** levels. VAYS contains no permission logic
of its own — it only shows or hides UI elements based on what YAC returns.

{: .note}
`GET /entity` intentionally returns the type metadata of **all** types to any
authenticated user (the UI needs it to render the navigation); the entity data
itself remains protected by roles.

## How file-level permissions are computed

For every request, YAC walks all [role definitions](file/roles.md) with keys of
the form `{type}:{set}:{perm}`:

- The `{type}` part must equal the requested entity type (there is no wildcard
  for types).
- The `{set}` part is either the literal `all` (matches every entity of the
  type) or the name of a [set](file/sets.md), whose test expression must render
  `true` for the entity. A role referencing an undefined set never matches.
- The role's own test expression (the value of the role key) must render
  `true`.

The `{perm}` part holds one or more permission flags joined with `+` (e.g.
`add+del`). The flags of all matching roles are collected and then *expanded*:
most write permissions imply `see` (see the table below).

Everything fails closed: if a set or role expression raises a template error,
the role does not match (and the error is logged). A user without any matching
role has no permissions at all on the entity.

Permission flags can be defined freely. Flags without a reserved meaning
(*custom perms*) have no built-in effect and imply nothing (in particular not
`see`), but can be required by [actions](file/types/actions.md) (via
`types[].actions[].perms`) and by subschemas (via
[`yac_perms`](file/schema.md#keyword-yac_perms)).

## Reserved permissions

| Perm  | Implies  | Description |
|:-----:|:--------:|:------------|
| `see` |          | Read the entity: its data, raw YAML and logs. Entities without `see` are also filtered out of the entity list. Reading is all-or-nothing per entity — [`yac_perms`](#property-level-permissions-yac_perms) never restricts it. |
| `add` | `see`    | Create a new entity of this type. |
| `rnm` | `see`    | Rename the entity **without** a revalidation of the YAML[^1]; additionally requires `add`. |
| `cpy` | `see`    | Copy the entity **without** a revalidation of the copied YAML; additionally requires `add`. |
| `lnk` | `see`    | Link to the entity **without** a revalidation of the linked YAML; additionally requires `add`. |
| `edt` | `see`    | Change the entity data (property-level `yac_perms` restrictions still apply). |
| `cln` | `see`    | "Clean": additionally required — *on top of* `edt` — for changes the schema does not cover[^2]: removing keys the schema does not define, and any change to comments, key order or formatting. `cln` alone does not allow any change. |
| `del` |          | Delete the entity. Does **not** imply `see`. |
| `act` |          | Execute actions[^3] (the default for an action's `perms` list; can be overridden per action). Does **not** imply `see`. |
| `all` | `see` + `add` + `rnm` + `cpy` + `lnk` + `edt` + `cln` + `del` + `act` | Shorthand for granting all of the above. `all` itself never ends up in the user's permission set, so it cannot be required in `yac_perms` or action `perms`! |
| `adm` | `all` + `adm` | Like `all`, but the flag `adm` itself *is* added to the permission set (so it can be required in `yac_perms` or action `perms`). It additionally unlocks the [admin override](#admin-override-force): writing an entity past a failing schema validation. |

## What each operation requires

| Operation | Required file-level permissions |
|:----------|:--------------------------------|
| List entities / read one / raw YAML / logs | `see` |
| Create (new)            | `add` |
| Create (copy)           | `add` + `cpy` |
| Create (link)           | `add` + `lnk` |
| Edit entity data        | `edt`, plus `cln` if the edit is "structural"[^2] |
| Rename (a change with a new name) | `add` + `rnm`[^1], plus `edt`/`cln` if the content changes too |
| Delete                  | `del` |
| Run an action           | at least **one** of the action's `perms` (default `[act]`)[^3] |

An edit that changes nothing (neither the content nor the name) is refused by
the write endpoints (`400`); this includes a `PATCH` whose values all equal
the stored ones. `POST /validate` accepts it from users holding
`edt` or `rnm` — this is how a client asks for the schema of an entity it is
about to edit.

`POST /validate` never returns anything derived from a stored entity (its
data, its YAML, a schema generated from it) to a user without `see` on that
entity, whatever the operation. It answers a request for an entity that does
not exist in the same way (unless one is to be created), so it does not tell
which names exist either, and it reports the usage of
[limits](file/types/limits.md) only if the operation is allowed. What every
authenticated user does get is the schema for *creating* an entity of a
type — like the type metadata, and including whatever the functions in the
schema return for that user: the form is rendered from it before the name
(and with it possibly the right to create) is settled. Do not put anything
into a schema that only some users may know.

An entity that other entities link to can neither be renamed nor deleted
(`400`, and `POST /validate` reports it): remove the links first.

### Rename, copy and link: two entities

A rename, copy or link involves the stored **source** entity and a **new
name**. The roles are therefore evaluated twice, and each permission is taken
from the evaluation it belongs to:

| Permission | Evaluated for | Variables |
|:-----------|:--------------|:----------|
| `rnm`, `cpy`, `lnk` — and for a rename also `edt`, `cln`, `adm`, the property-level [`yac_perms`](#property-level-permissions-yac_perms) and the `perms` of actions | the **source** entity on its own | `name` and `old.name` are the name of the source, `old.data` its data, `new.name` is unset |
| `add` (and, for a copy or link, the `perms` of actions) | the request as a whole, i.e. the **new name** | `name` and `new.name` are the new name, `old.data` is the data of the source |

So a role that is scoped on `name` (e.g. "all hosts in my subnets") lets its
holder rename, copy or link only entities whose *current* name is in that
scope — the new name being in scope is not enough. Rights that follow the
stored data (`old.data`) apply to the source and, as the data moves along, to
the new name as well. The source is evaluated in the same way when the name
of the copy or link is [generated](file/types/index.md#generated-names).

## Admin override (`force`)

The write endpoints (`POST /entity/{type}`, `PUT /entity/{type}/{name}`)
accept a `force=true` query parameter — the *admin override*. It requires the
`adm` permission (a request with `force` but without `adm` is rejected with
`403`, regardless of the data) and must be requested **explicitly**: holding
`adm` alone never bypasses anything.

With `force`, the entity is written even when its data fails **schema**
validation. Everything else is still enforced exactly as without it:

- the payload must be syntactically valid YAML,
- name rules (pattern, rename gates) apply,
- file-level permissions, conflict detection (`yaml_old`) and
  [limits](file/types/limits.md) apply.

Note that property-level [`yac_perms`](#property-level-permissions-yac_perms)
guards are **also bypassed**: they are enforced through the generated schema,
so a forced write can change values guarded even by *custom* perms the user
does not hold (`adm` expands to all reserved perms, not to custom ones).
Treat `adm` as the most powerful permission there is and grant it accordingly.

When a forced write actually bypassed a failing validation, the commit message
in the repository is suffixed with `(admin override: schema validation
bypassed)`, so the git history records every such write. A forced write whose
data happens to be valid is an ordinary commit.

`POST /validate` is unaffected — it always reports the real validation state
(this is what lets VAYS keep showing the errors while the override is
unlocked). It does however return the user's expanded permissions in `perms`,
so a UI can decide whether to offer the override at all.

{: .warning}
An override commit stores schema-invalid data in the repository. Consumers of
the repo must tolerate such files, and regular users editing the entity later
will see the migration-style validation errors until the data is fixed.

**In VAYS**, users holding `adm` always see a lock button next to the Commit
button when editing or creating. Unlocking it — after a warning dialog — lifts
the YAML editor's [locks on fixed values](../../vays/guarantees.md) and sends
the write with `force=true`; while the data fails validation, the Commit
becomes a red *Commit (admin)*. All validation errors stay visible while
unlocked.
The override is per-commit intent, not a mode: it locks again automatically
after one successful commit and whenever the edit view is (re)opened.

## Property-level permissions (`yac_perms`)

Every subschema in the [schema](file/schema.md) may carry a `yac_perms` list;
the user needs at least **one** of the listed perms to *write* the data
covered by that subschema.

- **Inheritance**: `yac_perms` applies recursively — the most specific (i.e.
  deepest) `yac_perms` on the path to a property wins. As soon as a subschema
  defines `yac_perms` explicitly, *nothing* is inherited from its parents
  (including `edt` from the top level). The top-level default is `[add, edt]`.
- **Write-side only**: `yac_perms` restricts what a user may *write* — it
  never hides data. Read access exists only at entity level: anyone holding
  `see` gets the **full** entity data, including the raw YAML; without `see`
  they get nothing at all. Consequently `see` (listed in or missing from) a
  `yac_perms` inside the schema has **no** read effect whatsoever — there is
  no way to read-protect single properties. On read operations, `yac_perms`
  is ignored entirely.
- **On create**, `add` is treated as held when evaluating `yac_perms`
  (creating the entity as a whole already required `add`). So properties whose
  `yac_perms` include `add` are writable during creation even by users who
  lack `edt`.
- **Enforcement**: subschemas the user may not write are removed from the
  generated JSON/UI schema (VAYS does not render them as editable). Stored
  values at removed paths are echoed back into the schema as read-only
  `const` properties, which also pins them: a request that changes or
  deletes such a value fails the schema validation (`400`), and setting a
  value where none is stored is rejected by the object's default
  `additionalProperties: false`.

{: .warning}
Explicitly setting `additionalProperties: true` on an object opts out of the
write enforcement for guarded properties **without a stored value** (stored
values remain pinned by their `const`). Keep objects closed — the default —
wherever `yac_perms` write gating must be airtight.

{: .warning}
All of this is done for objects that are declared as such: `type: object`,
also as one of several types (`type: [object, "null"]`). A restricted
property (`yac_perms`, `yac_editable`) in the `properties` of a subschema
without `type: object` could not be enforced and is therefore reported as an
error in the specs (`500`) as soon as it takes effect.

---

[^1]: A rename is submitted as a regular edit operation with a new name. A
      *pure* rename (no content change) moves the stored YAML as-is — the data
      is **not** revalidated against the schema, so entities whose stored data
      no longer matches the current schema can still be renamed. If the request
      also modifies the content, the edit is validated as usual (and
      `edt`/`cln` are required accordingly). Because a rename creates a new
      and deletes the old entity name, it additionally requires the
      type-level `create` **and** `delete`
      [switches](file/types/index.md#enabling-operations) to be enabled.

[^2]: "Structural" means anything the schema does not describe. Two things
      fall under that:

      - **Keys the schema does not define.** Stored keys without a subschema
        are echoed back into the generated schema as read-only `const`s; only
        `cln` holders may drop them. Keys the schema *does* define are
        governed by the schema alone: removing an optional one is an ordinary
        data change (`edt`), a key that vanished because its
        [`yac_if`](file/schema.md#keyword-yac_if) no longer holds *must* be
        removed, and a key guarded by
        [`yac_perms`](file/schema.md#keyword-yac_perms) / `yac_editable`
        stays immutable for the user — `cln` does not override that, and
        neither does a `yac_if` of that key that no longer holds.
        Inside arrays, such values can be moved between and removed with
        the items, see
        [Stored data without a schema](file/schema.md#stored-data-without-a-schema).
      - **Everything outside the data**: comments, the order of existing
        object keys, quoting and scalar style (like whether an *unchanged*
        value is split over multiple lines or not) and anchors. Blank lines
        are ignored. Inserting or removing a key is a data change, whatever
        position it has; a comment attached to a removed key goes with it.
        The order of list elements is data as well (the schema governs
        whether it may change).

      Only a raw-YAML edit (`PUT /entity/{type}/{name}`, VAYS's expert mode)
      can be structural: a data patch (`PATCH`, the VAYS form) is applied by
      YAC onto the stored YAML, which keeps comments, order and formatting by
      construction, so it never needs `cln`. VAYS's YAML editor locks the
      fixed values and (without `cln`) the comments, see
      [Locked Regions](../../vays/guarantees.md#locked-regions-in-the-yaml-editor).

[^3]: Only enforced when the action runs standalone (`arbitrary`) or is hooked
      without the `force` flag. Actions hooked to an operation with
      `force: true` always run with that operation, regardless of the action's
      `perms` — the operation's own permissions still apply, of course.
