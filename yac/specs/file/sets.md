---
parent: File
nav_order: 6
---

# Section `sets`

The sets that are referenced in your role definitions need to be defined in
this section. (That's also the only purpose of this section.) They are
evaluated at runtime for each (if needed).

A set **must not** be named `all`, as this set already exists implicitly
(see [Section `roles`](roles.md)).

## Examples

```yaml
sets:
  animal:
    zooanimals: "name in ['zebra', 'elephant', 'lion']"
    wildanimals: "name not in ['zebra', 'elephant', 'lion']"
```

{: .note}
To improove entity listing performance, split sets by entity-dependent
(using variables `old`, `new` or `name`) and *not* entity-dependent (using
any of the other variables). This allows YAC to optimize runtime by evaluating
all sets that are not entity-dependent once per request instead of once per
entity.

{: .warning}
Never use entity-variables (`old`, `new` or `name`) via context in custom
jinja2-functions, filters or tests directly. Always pass them explicitly!
Due to the performance optimization, YAC might otherwise assume the test
to be request-scoped and only run it once for multiple entities.
