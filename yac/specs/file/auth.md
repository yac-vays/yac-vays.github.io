---
parent: File
nav_order: 4
---

# Section `auth`

The `auth` section configures authentication (OpenID Connect) and CORS.
All values are **static** — changes require a pod/container restart.
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

## `oidc`

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
depending on what the IdP issues).

## `cors`

| Key       | Type            | Default                | Description |
|:----------|:----------------|:-----------------------|:------------|
| `origins` | `list[string]`  | `["https://localhost"]`| Allowed CORS origins. |

## Example with env injection

Pull the OIDC discovery URL out of `YAC_ENV__OIDC_URL` so the same specs
file can be used across environments:

{% raw %}
```yaml
auth:
  oidc:
    url: "{{ env.OIDC_URL }}"
    client_ids: ["{{ env.OIDC_CLIENT_ID }}"]
  cors:
    origins: ["{{ env.UI_ORIGIN }}"]
```
{% endraw %}
