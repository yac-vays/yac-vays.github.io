---
parent: Renderers
grand_parent: VAYS
nav_order: 19
---

# Renderer `random_string`

A text input that fills itself with a fresh random value — a UUID, a
hex token, an alphanumeric key, ... — generated in the user's browser.
Unlike [`age_secret`](age_secret.md), the value is stored (and shown)
in **plaintext**: it ends up as-is in the YAML file. Use it for
non-secret randomness like machine IDs, tokens that the target system
will read from the file anyway, correlation keys, and the like.

## Behaviour

  - **Set once.** When YAC supplies *neither a value nor a `default`*
    for the field, a random value is generated once and put into the
    form data. It is saved with the form like any typed-in value and
    never touched again on later edits (the value then comes from the
    YAC side).
  - **Generated at load time on create.** When creating a new entity,
    all `random_string` fields are filled while the form loads so the
    values are in the YAML document from the start. On *edit*, a
    field that is empty (e.g. added to the schema after the entity
    was created) is only filled when it is actually shown.
  - **Default wins.** If the schema has a `default`, nothing is
    generated — the field stays empty and the YAC-side default
    applies, as for any other string field.
  - **Editable.** The user can overwrite or clear the generated value
    by hand (clearing removes the key from the data). Validation
    (`pattern` etc.) is the usual YAC-side schema validation.
  - **Regenerate.** A **"Regenerate"** button replaces the current
    value with a new random one. If the value being replaced came from
    the YAC side (i.e. it existed before this editing session), a
    confirmation dialog is shown first; replacing a value generated or
    typed in the current session doesn't ask. Either way the change is
    only effective once the form is saved.

## Configuration

Select the renderer with `vays_options.renderer: random_string`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `string` (or omitted) for the tester to match. |
| `pattern`                                                | Recommended: mirror the configured format (e.g. `^[0-9a-f]{16}$` for `hex` of length 16) so hand-edited values are validated by YAC. |
| `default`                                                | If set, **suppresses generation** — the YAC-side default applies instead. |
| `vays_options.renderer_options.format`                   | One of `uuid`, `alphanumeric` (default), `hex`, `base64url`, `ascii_printable`, `custom`. See below. |
| `vays_options.renderer_options.length`                   | Length of the generated value. Default `32`. Ignored for `uuid`. |
| `vays_options.renderer_options.charset`                  | **Required for (and only used by) `format: custom`.** The literal alphabet to draw from, e.g. `"abcdef012345-"` (at most 256 distinct characters). |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — the renderer manages the value itself. |

### Formats

| `format`          | Generates |
|:------------------|:----------|
| `uuid`            | A random (version 4) UUID, e.g. `1f0e6f9c-64c5-4c5e-9bfa-1a3b1c2d3e4f`. |
| `alphanumeric`    | `length` characters from `A-Z a-z 0-9`. The default. |
| `hex`             | `length` lower-case hex digits (`0-9 a-f`). |
| `base64url`       | `length` characters from `A-Z a-z 0-9 - _`. |
| `ascii_printable` | `length` characters from letters, digits and most ASCII punctuation. |
| `custom`          | `length` characters from `renderer_options.charset`. |

The generator uses the browser's CSPRNG (`crypto.getRandomValues` /
`crypto.randomUUID`) with rejection sampling, so all alphabet
characters are equally likely.

An unknown `format`, or `custom` without a `charset`, is a spec error:
the field is disabled, marked invalid and reported in the
troubleshooting dropdown; nothing is generated.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    machine_id:
      title: Machine ID
      description: Stable random identifier, generated at creation time.
      vays_category: General
      type: string
      pattern: "^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$"
      vays_options:
        renderer: random_string
        renderer_options:
          format: uuid

    node_token:
      title: Node Token
      description: Random join token, readable in the YAML file.
      vays_category: General
      type: string
      pattern: "^[0-9a-f]{40}$"
      vays_options:
        renderer: random_string
        renderer_options:
          format: hex
          length: 40
```
{% endraw %}

For values that must not be readable in the YAML file, use
[`age_secret`](age_secret.md) instead.
