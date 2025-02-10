---
parent: File
nav_order: 6
---

# Section `context`

In this section, you can simple define some context variables that you want to
use in the schema for templating. Templating inside the `context` section is
not supported however, so the variables will be available 1:1 as they are
defined here.

## Example

{% raw %}
```yaml
context:
  restricted: false
  restricted_pattern: '^[a-z]{1,10}$'

schema:
  type: object
  properties:
    my_string:
      type: string
      pattern: "{{ context.restricted_pattern if context.restricted else omit }}"
```
{% endraw %}
