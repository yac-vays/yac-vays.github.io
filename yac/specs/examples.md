---
parent: Specification
nav_order: 4
---

# Examples

Examples that exercise the spec sections in increasing complexity. The
example field paths are linked to their reference in [File](file/index.md).

## Minimal

The smallest specs that does something useful — one type, one role, a
flat schema. The `repo` and `auth` blocks are static and used at process
startup; everything else may use [Jinja2 templating](j2.md).

{% raw %}
```yaml
types:
  - name: file
    title: Files

repo:
  plugin: git_direct
  connection:
    url: https://user:pass@git.example.com/my/repo.git
    branch: main
  details:
    file: "files/{{ name }}.yml"

auth:
  oidc:
    url: https://idp.example.com/.well-known/openid-configuration
    client_ids: [yac-spa]
  cors:
    origins: [https://yac.example.com]

roles:
  - file:all:all: "user.name == 'admin'"

schema:
  type: object
  properties:
    message:
      title: My Message
      vays_category: General
      type: string
      pattern: '^[a-zA-Z0-9 ]{1,100}$'
    worked:
      title: Did it work?
      vays_category: General
      type: boolean
      default: false
```
{% endraw %}

## Multiple Types, Sets and Roles

Two types, with different file layouts. Permissions differ per entity
subset (`sets`) and per group of users.

{% raw %}
```yaml
types:
  - name: animal
    title: Animals
    name_pattern: '^[a-z][a-z0-9-]{0,30}$'
    name_example: rex
  - name: cage
    title: Cages

repo:
  plugin: git_direct
  connection:
    url: https://user:pass@git.example.com/zoo.git
    branch: main
  details:
    animal: "animals/{{ name }}.yml"
    cage:   "cages/{{ name }}.yml"

sets:
  animal:
    zooanimals: "name in ['zebra', 'elephant', 'lion']"
    wildanimals: "name not in ['zebra', 'elephant', 'lion']"

roles:
  - animal:all:see:           "true"
  - animal:zooanimals:edt:    "'zoo-keepers' in user.token.groups | default([])"
  - animal:wildanimals:edt:   "'rangers' in user.token.groups | default([])"
  - animal:all:add+del:       "user.name == 'cleanup-script'"
  - cage:all:all:             "'zoo-keepers' in user.token.groups | default([])"

schema:
  type: object
  properties:
    color:
      title: Color
      vays_category: General
      type: string
      enum: [grey, brown, red]
      default: grey
```
{% endraw %}

## Templating, Conditionals and Optional Properties

Use [Jinja2 templating](j2.md), conditional defaults via the `if`
construct, and `yac_optional` to mark fields as not required.

{% raw %}
```yaml
context:
  max_age: 120

types:
  - name: course
    title: Courses
    name_generated: enforced
    name_generator: "new.data.title | lower | regex_replace(' ', '-')"

repo:
  plugin: git_direct
  connection:
    url: https://user:pass@git.example.com/courses.git
    branch: main
  details:
    course: "courses/{{ name }}.yml"

roles:
  - course:all:all: "user.name == old.data.responsible | default('')"
  - course:all:add: "'lecturers' in user.token.groups | default([])"

schema:
  type: object
  properties:

    title:
      title: Title
      vays_category: General
      type: string
      pattern: '^[A-Za-z0-9 .,_-]{5,100}$'

    responsible:
      title: Responsible
      vays_category: General
      type: string
      default: "{{ user.name }}"
      enum: "{{ [old.data.responsible | default(user.name), user.name] | unique | list }}"

    age:
      title: Maximum Age (years)
      yac_optional: true
      vays_category: General
      type: integer
      minimum: 0
      maximum: "{{ context.max_age }}"

    publish:
      title: Published
      vays_category: General
      type: boolean
      default: false

    publish_date:
      title: Publish Date
      yac_optional: true
      vays_category: General
      type: string
      format: date
      yac_if: "new.data.publish == true"
      default: "{{ now() | to_datestr('%Y-%m-%d') }}"
```
{% endraw %}

## Actions and Logs

Hook a custom action into the lifecycle and surface external logs in the
UI.

{% raw %}
```yaml
types:
  - name: host
    title: Hosts
    actions:
      - name: provision
        title: Provision
        description: Provision the host via the API.
        dangerous: true
        perms: [act, adm]
        hooks: [arbitrary, create:after]
        plugin: http
        details:
          method: POST
          url: "https://api.example.com/hosts/{{ name }}"
          headers:
            Authorization: "Bearer {{ env.api_token }}"
          success: [200, 202]
          error: [409]
    logs:
      - name: provisioning
        title: Provisioning Log
        progress: true
        problem: true
        plugin: file
        details:
          path: "/logs/hosts/{{ name }}.log"
          line_format: "[{time}] {progress} {message}"
          time: "log.time"
          message: "log.message"
          progress: "log.progress | int"
          problem: "log.message is regex_match('.*ERROR.*')"

repo:
  plugin: git_direct
  connection:
    url: https://user:pass@git.example.com/hosts.git
    branch: main
  details:
    host: "hosts/{{ name }}.yml"

roles:
  - host:all:all: "'ops' in user.token.groups | default([])"

schema:
  type: object
  properties:
    fqdn:
      title: FQDN
      vays_category: General
      type: string
      format: hostname
```
{% endraw %}

## See Also

  - [`request`](file/request.md) — custom HTTP headers
  - [`types`](file/types.md) — full reference for actions, logs, options
  - [`schema`](file/schema.md) — full list of supported keywords
  - [Permissions](perms.md) — what each `perm` actually allows
