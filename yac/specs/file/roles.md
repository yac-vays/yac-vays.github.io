---
parent: File
nav_order: 5
---

# Section `roles`

In this section, you can define permissions that are evaluated at runtime for
each entity individually (if needed).

Permission flags (`{perm}`) can be defined freely, but there are some predefined
ones: `see`, `add`, `rnm`, `cpy`, `lnk`, `edt`, `cln`, `del`, `act`, `all`, `adm`

For details, see: [Permissions](../perms.md)

Role keys can have the following two forms:

| Role Format           | Description |
|:----------------------|:------------|
| `{type}:all:{perm}`   | Grant user the `perms` on all entities of `type`. |
| `{type}:{set}:{perm}` | Grant user the `perms` on the entities of `type` in the set named `set` (see [Section `sets`](sets.md)). |

{: .note}
To improve entity listing **performance**, split roles by entity-dependent
(using variables `old`, `new` or `name`) and *not* entity-dependent (using
any of the other variables). This allows YAC to optimize runtime by evaluating
all roles that are not entity-dependent once per request instead of once per
entity.

{: .note}
For a rename, copy or link, the roles are evaluated twice: for the source
entity on its own and for the new name. `rnm`, `cpy` and `lnk` must be
granted for the **source** — see
[Permissions](../perms.md#rename-copy-and-link-two-entities).

{: .warning}
Never use entity-variables (`old`, `new` or `name`) via context in custom
jinja2-functions, filters or tests directly. Always pass them explicitly!
Due to the performance optimization, YAC might otherwise assume the test
to be request-scoped and only run it once for multiple entities.

## Examples

```yaml
roles:
  - animal:all:add+act: "user.name in old.data.responsible | default([])"
  - animal:all:del: "user.name == 'cleanup-script-user'"
  - animal:all:all+mycustomperm: "name in my_custom_lookup_func(user.name)"
  - animal:all:all+mycustomperm: "'admins' in user.token.ou"
  - animal:dogs:add+del: "user.name == 'dog-walker'"
  - animal:zooanimals:add+rnm+cpy+lnk+edt: "my_zooanimal_test_func(user.name)"
```

### Performance Optimization

In the following example, the LDAP lookup is done for every entity:

```yaml
roles:
  - animal:all:edt: "lookup_in_ldap_if_admin(user) or old.data.owner == user.name"
```

In the split-up version below, YAC can optimize by running the first one only
once per request instead of once per entity:

```yaml
roles:
  - animal:all:edt: "lookup_in_ldap_if_admin(user)"
  - animal:all:edt: "old.data.owner == user.name"
```

Permissions only add up, so YAC also skips the test of a role for an entity
if everything the role grants is granted already: by the roles without an
entity-dependent test, or by the roles listed before it that matched. In the
example above, the second role is never tested for the admins. For roles
that grant the same permissions, list the one that matches most often (or is
cheapest to test) first.

### Permissions in the Entity List

Testing the entity-dependent roles for every entity is what a list of many
entities spends most of its time on. So the permissions the entity list has
determined for an entity are reused for the next list requests of the same
user, for **5 minutes** ([`YAC_LIST_PERMS_TTL`](../../env.md), in seconds;
`0` turns the reuse off), as long as

  - the entity has the same name and the same YAML,
  - the user is the same: name, e-mail, the claims of the token (except the
    ones that change with every token, like `exp`), the IP address and the
    `yac-*` request headers, and
  - the roles that are not entity-dependent give the same result (they are
    still tested for every request).

A role or set test that only looks at `user`, `request`, `env`, `name` and
`old` therefore behaves exactly as without the reuse. One that also depends
on **something else** -- the current time, or a custom function that looks
something up (in a directory, by DNS, ...) *together with* `old` or `name` --
can be up to 5 minutes behind **in the entity list**: an entity shows up (or
disappears), and its permissions change there, that much later.

{: .note}
This only concerns which entities the list shows and the permissions it
reports for them. Every operation on an entity (reading it, validating,
writing, running an action) tests the roles anew, so nothing is ever
permitted on the grounds of a reused result.

If a role test fails to render (e.g. because a lookup timed out), the role
does not match, but that result is not kept: the next list request tests it
again.

The 5 minutes are a trade-off: long enough to cover going back and forth
between the list and an entity, short enough that a change outside of YAC
shows up in the list while one still expects it to. Set
`YAC_LIST_PERMS_TTL` lower if roles follow the time of day closely (e.g.
bookings that start on the minute), or to `0` if the list has to be exact at
any moment.
