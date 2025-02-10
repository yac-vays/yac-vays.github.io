---
parent: File
nav_order: 5
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
