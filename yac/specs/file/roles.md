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
| `{type}:{set}:{perm}` | Grant user the `perms` on the entities of `type` in the the set named `set` (see [Section `sets`](sets.md)). |

{: .note}
To improove entity listing **performance**, split roles by entity-dependent
(using variables `old`, `new` or `name`) and *not* entity-dependent (using
any of the other variables). This allows YAC to optimize runtime by evaluating
all roles that are not entity-dependent once per request instead of once per
entity.

{: .warning}
Never use entity-variables (`old`, `new` or `name`) via context in custom
jinja2-functions, filters or tests directly. Always pass them explicitly!
Due to the performance optimization, YAC might otherwise assume the test
to be request-scoped and only run it once for multiple entities.

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
