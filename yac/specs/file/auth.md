---
parent: File
nav_order: 1
---

# Section `auth`

The `auth` section configures authentication (OpenID Connect) and CORS.
For which templating variables are available where in this section,
see [Templating](../j2.md).

```yaml
auth:
  oidc:
    url: ...
    client_ids: [a, b]
    jwt:
      name: "{sub}"
      full_name: "{name}"
      full_name_fallback: "{given_name} {family_name}"
      email: "{email}"
      email_fallback: "{sub}@localhost"
    access_tokens:
      audiences: []
      algorithms: [RS256]
      subjects: []
      jwt:
        name: "{sub}"
        full_name: "{name}"
        full_name_fallback: "{sub}"
        email: "{email}"
        email_fallback: "{sub}@localhost"
      accounts: {}
  cors:
    origins: ["https://app.example.com"]
```

YAC accepts two kinds of bearer tokens, both validated statelessly:

  - **OIDC id-tokens** (the default) — issued to an interactive user,
    e.g. by VAYS. The `aud` must match one of `client_ids`.
  - **OIDC JWT access tokens** (RFC 9068, opt-in) — for machine clients
    (scripts, tools, other services) using the OAuth2 `client_credentials`
    grant. Enabled by configuring `access_tokens.audiences`; a bearer
    token whose `aud` matches one of those audiences is validated as an
    access token instead of an id-token.

## Key `oidc`

| Key                     | Type            | Default                                              | Description |
|:------------------------|:----------------|:-----------------------------------------------------|:------------|
| `url`                   | `string`        | `https://localhost/.well-known/openid-configuration` | URL of the OpenID Connect discovery document. |
| `client_ids`            | `list[string]`  | `[]`                                                 | Accepted `client_id`s. The first is used as the default in the Swagger UI. |
| `jwt.name`              | format-string   | `{sub}`                                              | Extracts the user's `name` from the JWT id-token. |
| `jwt.full_name`         | format-string   | `{name}`                                             | Extracts the user's `full_name`. |
| `jwt.full_name_fallback`| format-string   | `{given_name} {family_name}`                         | Fallback when `jwt.full_name` cannot be rendered (e.g. missing claim). |
| `jwt.email`             | format-string   | `{email}`                                            | Extracts the user's `email`. |
| `jwt.email_fallback`    | format-string   | `{sub}@localhost`                                    | Fallback when `jwt.email` cannot be rendered. |

The JWT format-strings use Python `str.format` substitution against the
validated id-token claims (so `{sub}`, `{preferred_username}`, etc. work
depending on what the IdP issues). You can see the values of a token via
YAC API call `GET /me`.

## Key `oidc.access_tokens` (machine clients)

| Key          | Type                   | Default    | Description |
|:-------------|:-----------------------|:-----------|:------------|
| `audiences`  | `list[string]`         | `[]`       | Accepted `aud` values for JWT access tokens. Empty (the default) disables access-token support entirely. |
| `algorithms` | `list[string]`         | `[RS256]`  | Accepted JWS signature algorithms. |
| `subjects`   | `list[string]`         | `[]`       | Optional allow-list of accepted `sub` values. Empty accepts any subject that passes the signature/`iss`/`exp`/`aud` checks. |
| `jwt.*`      | format-strings         | see below  | Same five format-strings as `oidc.jwt.*`, applied to the access-token claims. The only different default is `full_name_fallback: "{sub}"`, because `client_credentials` tokens carry no user claims. |
| `accounts`   | `map[string] -> object`| `{}`       | Static identities for machine clients, keyed by the token's `sub`. Each entry may set `name`, `full_name` and/or `email`; unset fields fall back to the `jwt.*` format-strings. |

A typical machine client obtains a token with the `client_credentials`
grant and uses it as a normal bearer token:

```sh
TOKEN=$(curl -s "https://idp.example.com/token" \
    -d "client_id=my-robot" -d "client_secret=$SECRET" \
    -d "grant_type=client_credentials" | jq -r .access_token)
curl -H "Authorization: Bearer $TOKEN" "https://yac.example.com/entity/host"
```

Requirements and caveats:

  - The IdP must issue **JWT** ("self-contained", RFC 9068) access tokens
    for the client — opaque reference tokens cannot be validated
    statelessly and are not supported.
  - The token's `aud` must contain one of the configured `audiences`.
    How the audience is set (per-client configuration, RFC 8707
    `resource` parameter, ...) depends on the IdP.
  - Since a `client_credentials` token identifies a client rather than a
    person, attach a friendly identity either in the IdP (custom `name` /
    `email` claims in the token — preferred, since it is centrally
    managed) or locally via `accounts`:

    ```yaml
    access_tokens:
      audiences: [yac-prod]
      accounts:
        my-robot:
          full_name: Nightly Sync Robot
          email: robot-owners@example.com
    ```

  - Roles in the *specs* match on the resulting user exactly as for
    interactive users, so a machine client's `name` (by default its
    `sub`) is what you grant permissions to.
  - Validation is stateless: a leaked token stays valid until `exp`.
    Keep access-token lifetimes short in the IdP; revoke by rotating the
    client secret.

## Key `cors`

| Key       | Type            | Default                | Description |
|:----------|:----------------|:-----------------------|:------------|
| `origins` | `list[string]`  | `["https://localhost"]`| Allowed CORS origins. |

## Example

To allow some configuration to be set via environment variables, you can
define your own env variables:

{% raw %}
```yaml
auth:
  oidc:
    url: "{{ env.oidc_url }}"
    client_ids: ["{{ env.oidc_client_id }}"]
  cors:
    origins: ["{{ env.vays_origin }}"]
```
{% endraw %}
