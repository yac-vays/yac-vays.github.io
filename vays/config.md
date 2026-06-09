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
| `production`              | `boolean` | yes      | When `false`, VAYS shows the **Schema Warnings** notification (the bell in the header) that surfaces schema-design hints. |
| `defaultEditorLayout`     | `"form" \| "yaml" \| "both"` | no | The editor layout shown the **first** time a user opens an entity: the form pane only, the YAML editor only, or both side by side. As soon as the user moves the divider (or collapses a pane) their choice is remembered in the browser (local storage) and overrides this. Defaults to `both`. |
| `color.primary`           | `#hex`    | no       | Primary theme colour. |
| `color.primaryHighlighted`| `#hex`    | no       | Hover/active variant of the primary colour. |
| `oidcConf.server`         | `string`  | yes      | The OIDC provider's discovery URL (`.../.well-known/openid-configuration`). Must match the YAC backend's [`auth.oidc.url`](../yac/specs/file/auth.md). |
| `oidcConf.clientID`       | `string`  | yes      | The OIDC client / audience for the SPA. Must be present in [`auth.oidc.client_ids`](../yac/specs/file/auth.md) of every backend. |
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
  "defaultEditorLayout": "both",
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
domain/{backend-name}/{type}/create/{name}
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
    [`auth.cors.origins`](../yac/specs/file/auth.md), otherwise the
    browser will block requests.
  - **Schema Warnings bell.** Set `"production": false` on test/staging
    instances to expose the schema-design warnings that help admins refine
    their UI-Schema. The flag only toggles that header notification — it is
    a UX/authoring cue, not a security feature, and does not change logging
    or hide the `/dev-info` page.
