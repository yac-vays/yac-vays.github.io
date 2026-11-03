---
parent: Renderers
grand_parent: VAYS
nav_order: 22
---

# Renderer `template_text`

Renders a **read-only text built from a template and the current form
data**, with a *Copy* button. Use it to show users a value that is
derived from what they enter — e.g. the final user name assembled from
the entity name and fields of the current array item — instead of only
explaining how it is built.

Like [`info_box`](info_box.md), the field never carries data: mark it
`yac_optional: true` and `not: {}` so YAC doesn't allow data in the
YAML for that property.

## Behaviour

  - **Live.** The text updates as the user types.
  - **Placeholders** in `{...}` are replaced with values from the form
    data:

    | Placeholder    | Resolves to |
    |:---------------|:------------|
    | `{key}`        | The property `key` of the object that contains this field — inside an array of objects, the *current item*. |
    | `{key/sub/0}`  | A nested value below that object (`/` separated, numbers index arrays). |
    | `{../key}`     | `..` walks up one level from that object (e.g. from an array item to the array). |
    | `{/key/sub}`   | Absolute from the data root ([JSON pointer](https://datatracker.ietf.org/doc/html/rfc6901): `~1` = `/`, `~0` = `~`). |

    Strings, numbers and booleans are substituted. A value that is
    missing, empty, `null`, an object or an array leaves the
    placeholder as it is (shown dimmed) and disables the *Copy*
    button until it is filled in.
  - **Jinja2 first.** Like the rest of the [Schema](../../yac/specs/file/schema.md),
    the template is rendered by YAC with Jinja2 before it reaches VAYS.
    Use this for everything that is not in the form data, e.g. the
    entity name (`name`). Single braces `{key}` pass through Jinja2
    unchanged; don't start a placeholder with `#` (`{#` opens a Jinja2
    comment).
  - **Not known yet.** To mark a part that cannot be shown yet (e.g.
    a generated name before the entity is created), let Jinja2 emit a
    placeholder that resolves to nothing, like `{random}`: it is shown
    dimmed and the value cannot be copied, so nobody copies a dummy.

## Configuration

Select the renderer with `vays_options.renderer: template_text`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | **Ignored** — usually omitted (any type matches). |
| `title`                                                  | Shown as the label. |
| `description`                                            | Shown behind the info button (Markdown). |
| `vays_options.renderer_options.template`                 | **Required.** The template text (see above). |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — the renderer never reads or writes its own data. |

## [Specs](../../yac/specs/index.md) Example

Show the user name of each access account of a bucket, assembled as
`{bucket}-{basename}-{permissions}`. The bucket name is the
(generated) entity name, so it is only known after the entity was
created — until then, a preview with a dimmed `{random}` part is
shown:

{% raw %}
```yaml
types:
  - name: bucket
    name_generator: "new.data.group ~ '-' ~ new.data.name ~ '-' ~ ((['abcdefghijklmnopqrstuvwxyz0123456789'] * 6) | map('random') | join)"
    name_generated: enforced

schema:
  type: object
  properties:

    users:
      title: Users
      type: array
      items:
        type: object
        properties:

          basename:
            title: Base Name
            type: string

          username:
            title: User Name
            description: Assembled as `{bucket}-{basename}-{permissions}`.
            yac_optional: true
            not: {}
            vays_options:
              renderer: template_text
              renderer_options:
                template: "{{ name or (new.data.group | default('{group}', true) ~ '-' ~ new.data.name | default('{name}', true) ~ '-{random}') }}-{basename}-{permissions}"

          permissions:
            title: Permissions
            type: string
            enum: [read, write, admin]
```
{% endraw %}
