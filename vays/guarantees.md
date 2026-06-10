---
parent: VAYS
nav_order: 4
---

# Guarantees and Limitations

VAYS is driven almost entirely by the
[JSON-Schema and UI-Schema delivered by YAC](../yac/specs/file/schema.md).
That gives spec authors a lot of
flexibility, but it also means the frontend can be asked to render shapes
it cannot represent well. This page is a non-exhaustive list of where
those limits sit.

## Layout Guidelines

These are guidelines for spec authors that VAYS can render *gracefully*.
Going beyond them won't break anything — but the user experience will
degrade, especially on smaller screens.

| Guideline                   | Soft limit | Notes |
|:----------------------------|:-----------|:------|
| Favorite actions per type   | **≤ 4**    | 5 is the grey zone. Move less-used actions out of `favorites` if you have more. |
| Columns in the entity list  | **≤ 8**    | Wider tables push UX off-screen on laptops; consider promoting fewer values to `options`. |
| Top-level categories (tabs) | **≤ ~8**   | More tabs become hard to scan; group related fields into the same category. |
| Form fields per category    | **≤ ~25**  | Use `vays_group` to break up long categories. |

## Schema Limitations

The frontend honours the YAC subset of JSON-Schema documented under
[Section `schema`](../yac/specs/file/schema.md). The most relevant
limitations from VAYS' point of view:

  - **`oneOf` / `allOf` / `anyOf`** at object-property level are not
    rendered as form controls. Use [`yac_if`](../yac/specs/file/schema.md#keyword-yac_if)
    or [`yac_types`](../yac/specs/file/schema.md#keyword-yac_types) to
    gate properties conditionally.
  - **`if` / `then` / `else`** is supported only for *changing value
    ranges* of an existing property (e.g. tightening a `pattern` or
    `enum`). Adding/removing properties or changing default values
    conditionally is not supported.
  - **Nested arrays.** [`vays_options`](../yac/specs/file/schema.md#keyword-vays_options)
    is not stackable: it works for an *array of objects*, but **not** for
    an array of objects containing another array of objects.
  - **`vays_category` / `vays_group`** can only be set on subschemas
    whose ancestors are all `object`s (or the immediate `items` of an
    array). Setting them deeper (inside `oneOf`, `if`, etc.) does not
    surface a form control.

## Validation

VAYS validates form data with `ajv` against the schema YAC sent. That
covers all keywords listed under
[Specs > `schema`](../yac/specs/file/schema.md). Format keywords (e.g.
`hostname`, `email`) are validated **only on the YAC backend**; if the
user submits an invalid format, the error appears after submit instead
of inline.

## Browser Support

VAYS is built against modern evergreen browsers (Chrome, Firefox,
Safari, Edge). Internet Explorer is not supported and never will be.

## Authentication

VAYS uses `authorization_code` with PKCE against the configured OIDC
provider. The redirect URI is **always** `https://<vays-host>/oauth2-redirect`
and must be registered with the provider for the configured `clientID`.
There is no built-in support for username/password flows.
