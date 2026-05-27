---
parent: Renderers
grand_parent: VAYS
nav_order: 3
---

# `age_secret` Renderer

The `age_secret` renderer generates a fresh random secret directly in
the user's browser, encrypts it with an [AGE](https://age-encryption.org)
recipient public key, and stores **only the ciphertext** in the form
data (and therefore in the YAML file on disk).

The user sees the cleartext exactly once, right after it is generated,
with a prompt to copy it. After saving, the cleartext is gone — the
field just shows `*` characters and a button to generate a new secret
that overwrites the current one.

## Behaviour

| State                                | What the user sees |
|:-------------------------------------|:-------------------|
| No value (create or change, enabled) | A new secret is generated automatically. The cleartext is shown read-only with a "Copy" button and a warning that it will not be shown again. |
| Existing valid value, enabled        | A masked input (`*` of `length` characters) and a **"Generate new"** button. Clicking it asks for confirmation and then replaces the secret. |
| Existing valid value, disabled       | A masked input (`*` of `length` characters). No buttons. |
| Existing value in an unexpected format | The raw value is shown read-only, the field is marked as invalid, and a **"Generate new"** button is offered. |
| No value, disabled                   | An empty read-only input. No buttons. |

The "Generate new" button opens a confirmation dialog. The new secret
only becomes effective once the form is saved — until then, the form
can be discarded to keep the previous value.

## Configuration

The renderer is selected explicitly via `vays_options.renderer:
age_secret`. Its own options live under
`vays_options.renderer_options`.

| Option            | Type      | Default        | Description |
|:------------------|:----------|:---------------|:------------|
| `age_public_key`  | `string`  | *(required)*   | The AGE recipient public key (`age1...`) the secret will be encrypted for. |
| `length`          | `integer` | `32`           | Length of the generated cleartext, and number of `*` characters shown when masking an existing value. |
| `charset`         | `enum`    | `alphanumeric` | One of `alphanumeric`, `hex`, `base64url`, `ascii_printable`. Controls the alphabet used by the random-secret generator. |

If `age_public_key` is missing or invalid, the renderer surfaces a
spec-author error and refuses to generate a value.

## Security notes

  - **The cleartext is shown exactly once.** It lives only in the
    user's browser tab and never leaves it. If the user navigates away
    before copying it, the cleartext is irrecoverable — they must
    generate a new one.
  - **The YAML file only ever contains the ciphertext** (an ASCII-armored
    AGE blob, `-----BEGIN AGE ENCRYPTED FILE-----` ... ). Decryption
    happens out of band (typically by an operator or a downstream tool
    such as SOPS) using the matching AGE identity.
  - **YAC cannot validate the ciphertext.** YAC has no access to the
    matching AGE identity, so it cannot decrypt the value or check that
    it round-trips. For a structural sanity check (armor markers,
    base64-decodable body, canonical AGE v1 header), set
    `format: age_secret` on the field — this is a bundled
    [schema-format plugin](../../yac/specs/file/schema.md#keyword-format)
    that rejects values that are obviously not AGE ciphertexts but
    still cannot prove the value is decryptable.
  - **Regenerating invalidates the previous secret.** Any downstream
    system that still uses the old secret must be rotated to the new
    one. The "Generate new" confirmation dialog explicitly warns about
    this.
  - The encryption is performed in the browser via the
    [`age-encryption`](https://www.npmjs.com/package/age-encryption)
    library; the AGE public key is the only piece of cryptographic
    material the spec needs to include.

## Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    db_password:
      title: Database Password
      description: |
        Auto-generated, encrypted to the cluster's AGE key.
        **Copy the cleartext before saving** — it is not recoverable.
      vays_category: Secrets
      type: string
      format: age_secret   # YAC-side structural validation (optional but recommended)
      vays_options:
        renderer: age_secret
        renderer_options:
          age_public_key: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
          length: 40
          charset: base64url
```
{% endraw %}
