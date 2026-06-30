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
| `logo`                    | `string`  | no       | URL of the sidebar logo image. Falls back to the built-in project logo. See [Branding](#branding-logo--favicon). |
| `favicon`                 | `string`  | no       | URL of the browser-tab favicon. Falls back to the built-in project favicon. See [Branding](#branding-logo--favicon). |
| `production`              | `boolean` | yes      | When `false`, VAYS shows the **Schema Warnings** notification (the bell in the header) that surfaces schema-design hints. |
| `defaultEditorLayout`     | `"form" \| "yaml" \| "both"` | no | The editor layout shown the **first** time a user opens an entity: the form pane only, the YAML editor only, or both side by side. As soon as the user moves the divider (or collapses a pane) their choice is remembered in the browser (local storage) and overrides this. Defaults to `both`. |
| `color`                   | `object`  | no       | Theme colours. Every field is optional; see [Theming](#theming-color). |
| `oidcConf.server`         | `string`  | yes      | The OIDC provider's discovery URL (`.../.well-known/openid-configuration`). Must match the YAC backend's [`auth.oidc.url`](../yac/specs/file/auth.md). |
| `oidcConf.clientID`       | `string`  | yes      | The OIDC client / audience for the SPA. Must be present in [`auth.oidc.client_ids`](../yac/specs/file/auth.md) of every backend. |
| `backends`                | `array`   | yes      | List of YAC instances this VAYS connects to (see below). |

Anything left unset falls back to the built-in defaults, which reproduce the
reference branding of [yac-vays.github.io](https://yac-vays.github.io/) (the
project logo/favicon and a brownish colour scheme).

### Branding (logo & favicon)

`logo` (the sidebar logo) and `favicon` (the browser-tab icon) each accept any
of the following:

  - **Inline SVG markup** — paste the `<svg>…</svg>` straight into the JSON.
    This is the simplest option: no extra file to host, and the image travels
    with the config. (A logo shown on the coloured sidebar usually wants a
    single-colour, e.g. white, SVG.)
  - a **same-origin path** served alongside the app, e.g. `/logo.svg`;
  - a **`data:` URI** with the image inlined; or
  - an **absolute URL** on another origin — in that case VAYS automatically
    adds that origin to the `img-src` of its Content-Security-Policy, so it is
    not blocked.

### Theming (`color`)

All colours are optional `#hex` values; omitting one keeps its default. A lot is
derived from `color.primary` automatically, so setting just that already gives a
coherent theme:

  - the primary **opacities** (used for subtle backgrounds and hovers) and the
    light **page tint**; and
  - the **dark-mode surfaces** (`box`, `boxAlt`, `boxInput`, `boxStroke`) and
    the dark page background/stroke — dark mode is rendered as a *dark version
    of your colour scheme* (a dark brown for the default brownish primary, a
    dark blue for an ETH-blue primary, …), not a fixed blue-grey.

Override any of the derived values explicitly when you need finer control:

| Key                        | Description |
|:---------------------------|:------------|
| `color.primary`            | Brand colour (sidebar, buttons, accents). |
| `color.primaryHighlighted` | Hover/active variant of the primary colour. |
| `color.danger`             | Colour for destructive actions and errors. |
| `color.box`                | Dark-mode component surface. |
| `color.boxAlt`             | Dark-mode rear/background surface. |
| `color.boxInput`           | Dark-mode input / hover surface. |
| `color.boxStroke`          | Dark-mode component border. |
| `color.light.font`         | Light-mode main text colour. |
| `color.light.fontInverted` | Light-mode text on a primary-coloured surface. |
| `color.light.fontReduced`  | Light-mode muted / secondary text. |
| `color.light.stroke`       | Light-mode border / divider colour. |
| `color.light.background`   | Light-mode page background. |
| `color.dark.*`             | The same five neutrals for dark mode. |

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
  "favicon": "/favicon.svg",
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

### Re-branding with an inline logo

To fully re-brand the instance you typically only need a logo and the two
primary colours — everything else (opacities, the light tint and the whole dark
theme) follows. The logo (and favicon) can be pasted straight into the JSON as
inline SVG, so there is no extra file to host:

```json
{
  "title": "Acme Self-Service",
  "logo": "<svg viewBox=\"0 0 120 40\" xmlns=\"http://www.w3.org/2000/svg\"><!-- single-colour (e.g. white) wordmark for the coloured sidebar --></svg>",
  "favicon": "<svg viewBox=\"0 0 32 32\" xmlns=\"http://www.w3.org/2000/svg\"><!-- square icon --></svg>",
  "color": {
    "primary": "#08407E",
    "primaryHighlighted": "#215CAF"
  },
  "production": true,
  "oidcConf": { "...": "..." },
  "backends": [ "..." ]
}
```

## URL Schema

For reference, VAYS uses the following client-side routes:

```
/                                       Login (OIDC)
/oauth2-redirect                        OAuth2 callback
/{backend-name}/{type}/                 Overview (list of entities)
/{backend-name}/{type}/create/{name}?   Create (or copy) an entity
/{backend-name}/{type}/edit/{name}?   Edit an entity
/{backend-name}/{type}/read/{name}?     Read an entity (read-only)
/error-page                             Error page
/dev-info                               Developer info
```

The overview route additionally supports filtering and paging via query
string and fragment:

```
domain/{backend-name}/{type}/?{filter-key}={filter-value}#{Page}.{NumPerPage}
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
