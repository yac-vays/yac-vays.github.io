---
parent: File
nav_order: 6
---

# Section: `sets`

The sets that are referenced in your role definitions need to be defined in
this section. (That's also the only purpose of this section.) They are
evaluated at runtime for each entity as well.

## Examples

```yaml
sets:
  animal:
    zooanimals: "name in ['zebra', 'elephant', 'lion']"
    wildanimals: "name not in ['zebra', 'elephant', 'lion']"
```

## Performance hint

A set whose expression does **not** reference any of the entity-dependent
variables `old`, `new` or `name` is evaluated once per request rather
than once per entity (the corresponding role is then dropped entirely if
the set evaluates to `false`). Combine this with the
[role-splitting hint](roles.md#performance-hint-prefer-split-conditions)
to keep listings cheap in repositories with many entities.
