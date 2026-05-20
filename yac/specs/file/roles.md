---
parent: File
nav_order: 5
---

# Section `roles`

In this section, you can define permissions that are evaluated at runtime for
each entity individually.

Permission flags (`{perm}`) can be defined freely, but there are some predefined
ones: `see`, `add`, `rnm`, `cpy`, `lnk`, `edt`, `cln`, `del`, `act`, `all`, `adm`

For details, see: [Permissions](../perms.md)

Role keys can have the following two forms:

    {type}:all:{perm}:       grant user the *perms* on all entities of *type*
    {type}:{set}:{perm}:     grant user the *perms* on all entities of *type* in the
                             the set named *set* (for sets: see next section)

## Examples

```yaml
roles:
  - animal:all:add+act: "user.name in old.responsible | default([])"
  - animal:all:del: "user.name == 'cleanup-script-user'"
  - animal:all:all+mycustomperm: "name in my_custom_lookup_func(user.name)"
  - animal:all:all+mycustomperm: "'admins' in user.token.ou"
  - animal:dogs:add+del: "user.name == 'dog-walker'"
  - animal:zooanimals:add+rnm+cpy+lnk+edt: "my_zooanimal_test_func(user.name)"
```

## Performance hint: prefer split conditions

When listing entities, YAC has to evaluate every role test against every
entity. As an optimization, a role test that does **not** reference any of
the entity-dependent variables `old`, `new` or `name` is evaluated only
once per request — if the result is `false`, the role is dropped before
the per-entity loop even starts.

This means a role like

```yaml
roles:
  - animal:all:edt: "'admins' in user.token.ou and old.data.owner == user.name"
```

is fully re-evaluated for every entity, because the whole expression
mixes a user-only check with an entity-dependent one. Splitting it into
two roles lets YAC short-circuit the admin case once per request:

```yaml
roles:
  - animal:all:edt: "'admins' in user.token.ou"
  - animal:all:edt: "old.data.owner == user.name"
```

The same applies to [set definitions](sets.md): a set whose expression is
user-only is evaluated once per request rather than per entity.

For repositories with thousands of entities and many roles, this can
turn the cost of a list-entities call from "evaluate every role for
every entity" into "evaluate every role once, then evaluate only the
surviving entity-dependent residuals per entity".
