---
parent: Renderers
grand_parent: VAYS
nav_order: 2
---

# Special Renderers

These renderers are *only* picked when the YAC spec explicitly requests
them via `vays_options.renderer: <name>`. Each row tells you the required
JSON-Schema shape and the options it accepts.

| `renderer` value   | JSON-Schema requirement              | Purpose |
|:-------------------|:-------------------------------------|:--------|
| `info_box`         | `type: string` (or untyped)          | Render a read-only info box (uses `title` / `description`, no input). |
| `password`         | `type: string` (or untyped)          | Password input that hashes the value before sending. |
| `ssh_key`          | `type: string` (or untyped)          | Multi-line SSH-key input with file-paste support. |
| `text_area`        | `type: string` (or untyped)          | Multi-line text area instead of a single-line input. |
| `mac_address`      | `type: string` (or untyped)          | Text input with MAC-address-aware formatting. |
| `list_as_string`   | `type: string` (or untyped)          | Edit a list of items, but persist as a separator-joined string. |
| `big_string_list`  | `type: array, items: { type: string }` | High-performance list editor for long arrays of strings. |
| `multi_checkbox`   | `type: array` with `oneOf` items     | Render the choices as a column of checkboxes instead of a multi-select. |
| `age_secret`       | `type: string` (or untyped)          | Generate-once secret, AGE-encrypted in the browser before storage. See its [dedicated page](age_secret.md). |

## `password` options

The password is hashed in the browser **before** being sent over the
wire (and before it lands in the YAML file).

| Option              | Type     | Default          | Description |
|:--------------------|:---------|:-----------------|:------------|
| `save_password_as`  | `enum`   | `crypt-sha-512`  | One of `crypt-sha-512` (Unix-`crypt`-style SHA-512 hash, like `/etc/shadow`) or `plaintext`. Only use `plaintext` if the storing system genuinely needs the cleartext value. |

## `list_as_string` options

| Option       | Type     | Default | Description |
|:-------------|:---------|:--------|:------------|
| `separator`  | `string` | `,`     | Separator inserted between items when joining the list back into a string. |

## `text_area` options

| Option   | Type      | Default | Description |
|:---------|:----------|:--------|:------------|
| `rows`   | `integer` | -       | Visible row count of the text area. |

## `info_box`, `ssh_key`, `mac_address`, `big_string_list`, `multi_checkbox`

No renderer-specific options beyond the [common options](./#common-options).

## Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    welcome:
      title: Welcome
      description: |
        Please fill in **all** fields. See the [docs](https://example.com)
        if anything is unclear.
      vays_category: General
      type: string
      vays_options:
        renderer: info_box

    password:
      title: Password
      vays_category: Account
      type: string
      vays_options:
        renderer: password
        save_password_as: crypt-sha-512

    ssh_key:
      title: SSH Public Key
      vays_category: Account
      type: string
      pattern: '^(ssh-rsa|ssh-ed25519|ecdsa-sha2-[a-z0-9-]+) [A-Za-z0-9+/=]+( .+)?$'
      vays_options:
        renderer: ssh_key

    notes:
      title: Notes
      vays_category: General
      type: string
      vays_options:
        renderer: text_area
        rows: 8

    aliases:
      title: Aliases (comma-separated)
      vays_category: General
      type: string
      vays_options:
        renderer: list_as_string
        separator: ','

    tags:
      title: Tags
      vays_category: General
      type: array
      items:
        type: string
      vays_options:
        renderer: big_string_list
```
{% endraw %}
