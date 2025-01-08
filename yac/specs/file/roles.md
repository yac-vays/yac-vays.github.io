---
parent: File
nav_order: 3
---

# Section `roles`

In this section, you can define permissions that are evaluated at runtime for
each entity individually.

Permission flags (`{perm}`) can be defined freely, but there are some predefined
ones: `see`, `add`, `rnm`, `cpy`, `lnk`, `edt`, `cln`, `del`, `act`, `all`, `adm`

For details, see: [Permissions](../perms.md)

Role keys can have three different forms:

    all:{type}:{perm}:        grant user the *perms* on all entities of *type*
    {type}:{name}:{perm}:     grant user the *perms* on the entity of *type* named *name*
    set:{type}:{name}:{perm}: grant user the *perms* on all entities of *type* in the
                              set named *name* (for sets: see next section)

## Examples

```yaml
roles:
  - all:animal:add+act: "user.name in old.responsible | default([])"
  - all:animal:del: "user.name == 'cleanup-script-user'"
  - all:animal:all+mycustomperm: "name in my_custom_lookup_func(user.name)"
  - all:animal:all+mycustomperm: "'admins' in user.token.ou"
  - animal:dog:add+del: "user.name == 'dog-walker'"
  - set:animal:zooanimals:add+rnm+cpy+lnk+mod: "my_zooanimal_test_func(user.name)"
```
