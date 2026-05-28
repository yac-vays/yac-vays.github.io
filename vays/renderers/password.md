---
parent: Renderers
grand_parent: VAYS
nav_order: 11
---

# Renderer `password`

The `password` renderer is a masked text input that **hashes the
value in the browser before transmitting it**, so the cleartext
password never leaves the user's tab and never lands in the YAML
file on disk.

After the user types a password, the renderer debounces input
briefly and then writes a SHA-512 Unix `crypt`-style hash (the same
shape as `/etc/shadow` lines) into the form data. The displayed
value is masked with `*` characters; the actual cleartext is held
only in component state.

In `crypt-sha-512` mode (the default), the renderer also shows a
small "transmitted as SHA-512 UNIX crypt-hash" hint with a link to
the algorithm spec, so end users understand the value cannot be
read back.

## Behaviour

| State                        | What the user sees |
|:-----------------------------|:-------------------|
| No value (create or change)  | An empty password input with the field's `title` and `description`. |
| Existing value, enabled      | A masked input of 10 `*` (or the password's length when stored as `plaintext`). Typing replaces the value. |
| Existing value, disabled     | Same masking; the field is read-only. |

## Configuration

Select the renderer with `vays_options.renderer: password`.

| Keyword                                                       | Effect |
|:--------------------------------------------------------------|:-------|
| `type`                                                        | Must be `string` (or omitted) for the tester to match. |
| `format`                                                      | Recommended: `unix_password_hash` for YAC-side structural validation of the stored hash. |
| `pattern`                                                     | Validated against the **stored hash**, not the cleartext (in `crypt-sha-512` mode). Use it for hash-shape checks, not for password-strength policy. |
| `minLength` / `maxLength`                                     | Validated against the **stored hash**, not the cleartext (in `crypt-sha-512` mode). |
| `vays_options.renderer_options.save_password_as`              | Either `crypt-sha-512` (Unix-`crypt`-style SHA-512 hash, like `/etc/shadow`, default) or `plaintext` (write the cleartext verbatim — only use when the storing system genuinely cannot consume a hash). |
| `vays_options.initial` / `vays_options.initial_editable`      | **Ignored** — the renderer drives both internally to keep an existing value masked and never editable as a placeholder. Use JSON-Schema `default` if you need to seed the field. |

## Security notes

  - **The cleartext never leaves the browser** in `crypt-sha-512` mode.
    Hashing happens client-side; only the hash is sent to YAC and
    written to YAML.
  - **Pair with a schema format check.** Set `format: unix_password_hash`
    on the field to have YAC structurally validate that the stored
    value is a Unix-`crypt` SHA-256/SHA-512 or Bcrypt hash — see the
    [bundled formats](../../yac/specs/file/schema.md#keyword-format).
  - **`plaintext` defeats the renderer's purpose.** Only set
    `renderer_options.save_password_as: plaintext` when the storing
    system genuinely cannot consume a hash. The cleartext is then
    written verbatim to the YAML file.
  - The renderer does **not** enforce password strength. Use JSON-Schema
    `minLength` / `pattern` if you need policy enforcement.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    admin_password:
      title: Admin Password
      description: |
        Stored as a SHA-512 Unix-crypt hash. The cleartext is never
        transmitted or persisted.
      vays_category: Account
      type: string
      format: unix_password_hash   # structural validation on YAC's side
      minLength: 12
      vays_options:
        renderer: password
        renderer_options:
          save_password_as: crypt-sha-512
```
{% endraw %}
