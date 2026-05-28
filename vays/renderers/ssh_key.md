---
parent: Renderers
grand_parent: VAYS
nav_order: 12
---

# Renderer `ssh_key`

The `ssh_key` renderer is a specialised text input for OpenSSH
public keys. Compared to the plain text renderer it offers:

  - **Compact display.** Once a key is stored it is shown as
    `<key-type> (<comment>): <first 20 chars of the key body>...`,
    so long keys don't take over the form.
  - **File picker.** A paper-clip button next to the field opens a
    file picker that accepts `.pub` files and pastes their trimmed
    contents into the field.
  - **Edit button.** Switches back to a raw text mode so the user
    can paste or type the key directly. Pressing *Enter* or clicking
    outside freezes the value back into compact display.
  - **Copy button.** Copies the key to the clipboard.

If the stored value contains newlines, each line is rendered as its
own SSH-key row — useful when a field stores multiple keys joined
by `\n`.

## Configuration

Select the renderer with `vays_options.renderer: ssh_key`.

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `type`                           | Must be `string` (or omitted) for the tester to match. |
| `format`                         | Recommended: `ssh_key` for YAC-side structural validation of the OpenSSH public-key shape (see [bundled formats](../../yac/specs/file/schema.md#keyword-format)). |
| `pattern`                        | Non-matching values are reported inline as an error. |
| `minLength`                      | Too-short values are reported inline as an error (useful for required keys). |
| `vays_options.initial`           | **Repurposed** — sets the file-picker label (the field itself displays a compact summary of the stored key, not the placeholder). |
| `vays_options.initial_editable`  | When `true` together with `initial`, the value is loaded into the form data so the user starts from an existing key. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    ssh_key:
      title: SSH Public Key
      description: Paste your `id_*.pub` or pick the file with the picker.
      vays_category: Account
      type: string
      format: ssh_key
      vays_options:
        renderer: ssh_key
```
{% endraw %}
