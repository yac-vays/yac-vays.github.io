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

The field holds **exactly one** SSH key. If a selected file contains
multiple keys or is otherwise malformed, it is **not loaded** and an
inline error is shown instead. The same applies to a stored value that
is not a single, well-formed key.

## Configuration

Select the renderer with `vays_options.renderer: ssh_key`.

| Keyword                          | Effect |
|:---------------------------------|:-------|
| `type`                           | Must be `string` (or omitted) for the tester to match. |
| `format`                         | Recommended: `ssh_key` for YAC-side structural validation of the OpenSSH public-key shape (see [bundled formats](../../yac/specs/file/schema.md#keyword-format)). |
| `pattern`                        | Non-matching values are reported inline as an error. |
| `minLength`                      | Too-short values are reported inline as an error (useful for required keys). |
| `vays_options.initial`           | Behaves as [documented](../../yac/specs/file/schema.md#keyword-vays_optionsinitial), with one caveat regarding what "interacts" means (see *Warning* below). |
| `vays_options.initial_editable`  | When `true`, the `initial` value is pre-loaded as editable content — indistinguishable from a `default` for the user — exactly like in the text renderer. |

{: .note}
A value that is not a single, valid OpenSSH public key (e.g. it contains
multiple keys) is never loaded into the field; the renderer shows an
inline error so the malformed value cannot be silently kept.

{: .warning}
What counts as *interaction* is slightly more eager here than in the text
renderer. Because the field commits its value when it leaves edit mode
(on *Enter*, on clicking outside, or after a file load), clicking the
**Edit** button and then clicking away — *without changing anything* —
already commits the displayed `initial`/`default` value into the data. The
plain text renderer only commits once the text is actually changed.

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
