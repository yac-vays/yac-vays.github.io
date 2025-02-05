---
parent: File
nav_order: 4
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
  - animal:zooanimals:add+rnm+cpy+lnk+mod: "my_zooanimal_test_func(user.name)"
```
