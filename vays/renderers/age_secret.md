---
parent: Renderers
grand_parent: VAYS
nav_order: 18
---

# Renderer `age_secret`

The `age_secret` renderer generates a fresh random secret directly in
the user's browser, encrypts it with an [AGE](https://age-encryption.org)
recipient public key, and stores **only the ciphertext** in the form
data (and therefore in the YAML file on disk).

The user sees the cleartext exactly once, right after it is
generated, with a prompt to copy it. After saving, the cleartext is
gone — the field just shows `*` characters and a button to generate
a new secret that overwrites the current one.

## Behaviour

| State                                  | What the user sees |
|:---------------------------------------|:-------------------|
| No value (create or change, enabled)   | A new secret is generated automatically. The cleartext is shown read-only with a "Copy" button and a warning that it will not be shown again. |
| Existing valid value, enabled          | A masked input (`*` of `length` characters) and a **"Generate new"** button. Clicking it asks for confirmation and then replaces the secret. |
| Existing valid value, disabled         | A masked input (`*` of `length` characters). No buttons. |
| Existing value in an unexpected format | The raw value is shown read-only, the field is marked as invalid, and a **"Generate new"** button is offered. |
| No value, disabled                     | An empty read-only input. No buttons. |

The "Generate new" button opens a confirmation dialog. The new
secret only becomes effective once the form is saved — until then,
the form can be discarded to keep the previous value.

If `age_public_key` is missing or invalid, the renderer surfaces a
spec-author error and refuses to generate a value.

## Configuration

Select the renderer with `vays_options.renderer: age_secret`.

| Keyword                                                  | Effect |
|:---------------------------------------------------------|:-------|
| `type`                                                   | Must be `string` for the tester to match. |
| `format`                                                 | Recommended: `age_secret` for YAC-side structural validation (see [bundled formats](../../yac/specs/file/schema.md#keyword-format)). |
| `vays_options.renderer_options.age_public_key`           | **Required.** The AGE recipient public key (`age1...`) the secret will be encrypted for. |
| `vays_options.renderer_options.length`                   | Length of the generated cleartext, and number of `*` characters shown when masking an existing value. Default `32`. |
| `vays_options.renderer_options.charset`                  | One of `alphanumeric` (default), `hex`, `base64url`, `ascii_printable`. Controls the alphabet used by the random-secret generator. |
| `vays_options.initial` / `vays_options.initial_editable` | **Ignored** — the renderer always manages the value itself (auto-generating a fresh secret when the field is empty). |

## Security notes

  - **YAC has no way to validate** the encrypted value, because it
    does not (and must not) have the AGE private key. In theory, a
    user could encrypt and send any value (including an empty
    string, code injection attempts, ...)! So the target system that
    decrypts the value has to make sure it is safe to use!
  - However, what YAC can do is a structural sanity check (armor
    markers, base64-decodable body, canonical AGE v1 header). Set
    `format: age_secret` on the field to use the according
    [schema-format plugin](../../yac/specs/file/schema.md#keyword-format).
  - **The cleartext only lives in the user's browser memory.** It is
    shown until the user saves. Then, VAYS encrypts it with the
    public key and only sends the ciphertext (an ASCII-armored AGE
    blob, `-----BEGIN AGE ENCRYPTED FILE-----` ... ) to YAC's API
    (and thus the YAML file).
  - The encryption is performed via the
    [`age-encryption`](https://www.npmjs.com/package/age-encryption)
    library.

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    db_password:
      title: Database Password
      description: |
        Auto-generated, encrypted to the cluster's AGE key.
      vays_category: Secrets
      type: string
      format: age_secret
      vays_options:
        renderer: age_secret
        renderer_options:
          age_public_key: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
          length: 40
          charset: base64url
```
{% endraw %}
