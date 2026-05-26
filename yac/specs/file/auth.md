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
      name: "{name}"
      full_name: "{givenName} {surname}"
      full_name_fallback: "{name}"
      email: "{mail}"
      email_fallback: "{name}@localhost"
  cors:
    origins: ["https://app.example.com"]
```

## Key `oidc`

| Key                     | Type            | Default                                              | Description |
|:------------------------|:----------------|:-----------------------------------------------------|:------------|
| `url`                   | `string`        | `https://localhost/.well-known/openid-configuration` | URL of the OpenID Connect discovery document. |
| `client_ids`            | `list[string]`  | `[]`                                                 | Accepted `client_id`s. The first is used as the default in the Swagger UI. |
| `jwt.name`              | format-string   | `{name}`                                             | Extracts the user's `name` from the JWT id-token. |
| `jwt.full_name`         | format-string   | `{givenName} {surname}`                              | Extracts the user's `full_name`. |
| `jwt.full_name_fallback`| format-string   | `{name}`                                             | Fallback when `jwt.full_name` cannot be rendered (e.g. missing claim). |
| `jwt.email`             | format-string   | `{mail}`                                             | Extracts the user's `email`. |
| `jwt.email_fallback`    | format-string   | `{name}@localhost`                                   | Fallback when `jwt.email` cannot be rendered. |

The JWT format-strings use Python `str.format` substitution against the
validated id-token claims (so `{sub}`, `{preferred_username}`, etc. work
depending on what the IdP issues). You can see the values of a token via
YAC API call `GET /me`.

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
