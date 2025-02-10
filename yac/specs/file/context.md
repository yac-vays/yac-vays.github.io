---
parent: File
nav_order: 2
---

# Section `context`

In this section, you can simple define some context variables that you use in
different places for templating. Templating is not supported, so the variables
defined here will be available 1:1 as they are defined here.

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
