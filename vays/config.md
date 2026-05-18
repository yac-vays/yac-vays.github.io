---
parent: VAYS
nav_order: 2
---

# Configuration

VAYS itself needs very little configuration. The vast majority of UI
behaviour is driven by data delivered by the [YAC](../yac/index.md)
backends at runtime — most importantly the JSON-Schema and UI-Schema
generated from the [specs file](../yac/specs/index.md).

What VAYS *does* need is a small `config.json` file, served at
`/config.json` from the same origin as the app. It is fetched once on
start-up and tells VAYS which YAC backends to talk to, how to
authenticate, and what to call itself.

## Reference

| Key                       | Type      | Required | Description |
|:--------------------------|:----------|:---------|:------------|
| `title`                   | `string`  | yes      | Window title and header label of the VAYS instance. |
| `logo`                    | `string`  | no       | URL of an image used as the sidebar logo. Falls back to a built-in logo. |
| `production`              | `boolean` | yes      | When `false`, VAYS shows a "non-production" banner so users know they are on a test instance. |
| `color.primary`           | `#hex`    | no       | Primary theme colour. |
| `color.primaryHighlighted`| `#hex`    | no       | Hover/active variant of the primary colour. |
| `oidcConf.server`         | `string`  | yes      | The OIDC provider's discovery URL (`.../.well-known/openid-configuration`). Must match the YAC backend's [`YAC_OIDC_URL`](../yac/env.md). |
| `oidcConf.clientID`       | `string`  | yes      | The OIDC client / audience for the SPA. Must be present in [`YAC_OIDC_CLIENT_IDS`](../yac/env.md) of every backend. |
| `backends`                | `array`   | yes      | List of YAC instances this VAYS connects to (see below). |

### `backends[]`

| Key      | Type     | Required | Description |
|:---------|:---------|:---------|:------------|
| `name`   | `string` | yes      | URL-safe slug used in VAYS routes (e.g. `domain/<name>/<type>`). Must be unique. |
| `title`  | `string` | yes      | Human-readable name shown in the sidebar/backend switcher. |
| `icon`   | `string` | yes      | An inline SVG (or any HTML allowed in an `innerHTML` context) used as the backend's icon. |
| `url`    | `string` | yes      | Base URL of the YAC backend's API. |

## Example

```json
{
  "title": "Linux Client Self-Service",
  "logo": "/logo.svg",
  "color": {
    "primary": "#00596D",
    "primaryHighlighted": "#007894"
  },
  "production": true,
  "oidcConf": {
    "server": "https://access.example.com/.well-known/openid-configuration",
    "clientID": "vays-spa"
  },
  "backends": [
    {
      "name": "hosts",
      "title": "Hosts Inventory",
      "icon": "<svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24\" viewBox=\"0 -960 960 960\" width=\"24\"><path d=\"M40-120v-80h880v80H40Zm120-120q-33 0-56.5-23.5T80-320v-440q0-33 23.5-56.5T160-840h640q33 0 56.5 23.5T880-760v440q0 33-23.5 56.5T800-240H160Z\"/></svg>",
      "url": "https://yac-hosts.example.com"
    },
    {
      "name": "courses",
      "title": "Git Courses",
      "icon": "<svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24\" viewBox=\"0 -960 960 960\" width=\"24\"><path d=\"M200-120q-33 0-56.5-23.5T120-200v-560q0-33 23.5-56.5T200-840h560q33 0 56.5 23.5T840-760v560q0 33-23.5 56.5T760-120H200Z\"/></svg>",
      "url": "https://yac-courses.example.com"
    }
  ]
}
```

## URL Schema

For reference, VAYS uses the following client-side routes:

```
domain/{backend-name}/{type}/?{filter-key}={filter-value}#{Page}.{NumPerPage}
domain/{backend-name}/{type}/create/{name}?view=...
```

The `{backend-name}` segment matches `backends[].name` from the config.

## Common Pitfalls

  - **OIDC redirect URI mismatch.** VAYS uses
    `https://<host-of-vays>/oauth2-redirect` as the redirect URI. Register
    that exact URL with your OIDC provider for the `clientID` configured
    here.
  - **Mixing OIDC clients.** VAYS and YAC must both accept the same
    `clientID`; otherwise YAC will reject the access token.
  - **CORS.** The YAC backend must include the VAYS origin in its
    [`YAC_CORS_ORIGINS`](../yac/env.md), otherwise the browser will block
    requests.
  - **Production banner.** Set `"production": true` only on instances
    where it is genuinely safe to do so — the banner is a deliberate UX
    cue, not a security feature.
