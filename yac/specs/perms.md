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
| `cln` | `see`    | "Clean": additionally required — *on top of* `edt` — for changes that touch anything the schema does not cover[^2]. `cln` alone does not allow any change. |
| `del` |          | Delete the entity. Does **not** imply `see`. |
| `act` |          | Execute actions[^3] (the default for an action's `perms` list; can be overridden per action). Does **not** imply `see`. |
| `all` | `see` + `add` + `rnm` + `cpy` + `lnk` + `edt` + `cln` + `del` + `act` | Shorthand for granting all of the above. `all` itself never ends up in the user's permission set, so it cannot be required in `yac_perms` or action `perms`! |
| `adm` | `all` + `adm` | Like `all`, but the flag `adm` itself *is* added to the permission set (so it can be required in `yac_perms` or action `perms`). Its intended meaning — freely edit the entity data without validation — is **not implemented yet** and will only be on real demand. |

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

---

[^1]: A rename is submitted as a regular edit operation with a new name. A
      *pure* rename (no content change) moves the stored YAML as-is — the data
      is **not** revalidated against the schema, so entities whose stored data
      no longer matches the current schema can still be renamed. If the request
      also modifies the content, the edit is validated as usual (and
      `edt`/`cln` are required accordingly).

[^2]: "Structural" means anything the schema does not describe: deleting
      object keys / data, comments, syntax (like whether a value is split over
      multiple lines or not), spacing and the order of object keys — but not
      the order of list elements (that is part of the data, and the schema
      governs whether it may change). Data whose subschema was removed due to
      missing perms or conditions counts as not covered as well.

[^3]: Only enforced when the action runs standalone (`arbitrary`) or is hooked
      without the `force` flag. Actions hooked to an operation with
      `force: true` always run with that operation, regardless of the action's
      `perms` — the operation's own permissions still apply, of course.
