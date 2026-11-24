---
parent: VAYS
nav_order: 5
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
    rendered as form controls (except a `oneOf` of `const` values, see the
    [`one_of_enum`](renderers/one_of_enum.md) renderer). Use [`yac_if`](../yac/specs/file/schema.md#keyword-yac_if)
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
`hostname`, `email`) are validated **only on the YAC backend**; such an error is shown at the
field once the backend has validated the data, not instantly while typing.

What is committed is the document as YAC validated it last. Commit therefore
waits for running validations; if one is still running after 15 seconds,
nothing is written and an error is shown — commit again once the validation
has finished.

## Locked Regions in the YAML Editor

The YAML editor (expert mode) locks what the user cannot change anyway, so
they learn it right away instead of from the next validation. Locked parts
have a lock in the gutter, greyed text and an explanation on hover; typing
there is refused with Monaco's read-only message, and any other edit that
would change them (paste, shortcuts, a further cursor, find and replace,
joining lines) is taken back right away:

  - **Fixed values**: every value whose schema is a `const` — the only
    allowed value. YAC pins stored values the user may not write that way
    (missing [`yac_perms`](../yac/specs/file/schema.md#keyword-yac_perms),
    `yac_editable: false`, keys the schema does not define), spec authors
    can also write a `const`. A required one can neither be changed nor
    removed; an optional one (e.g. a key the schema does not define, for
    users with the [`cln`](../yac/specs/perms.md) permission) can't be
    changed either, but may be removed as a whole (select its lines and
    delete them) — it has a lighter lock. Every line of a multi-line value
    gets a lock. Not locked while the admin override is on (users with
    `adm` can unlock it next to the Commit button at any time).
  - **Comments**, for users without the `cln` permission: YAC refuses any
    change of comments without it, so their text cannot be edited, removed
    or added (`#` that would start a comment, or the editor's comment
    toggle). A key can still be deleted together with its inline comment
    (whole lines), as YAC allows that.

At the edges of a locked part the cursor is free — e.g. Enter after a fixed
value or a comment adds a new line. Blank lines and indentation are never
locked: YAC does not need `cln` for them.
Changes of key order or quoting, which also need `cln`, are not locked in
advance and are reported by the validation. While the YAML can't be parsed,
fixed values are not locked; comments still are. On a line with another indicator (an error, the
removed-lines bar, a limit dot), that indicator is shown instead of the lock.
The form is not affected.

## Browser Support

VAYS is built against modern evergreen browsers (Chrome, Firefox,
Safari, Edge). Internet Explorer is not supported and never will be.

## Authentication

VAYS uses `authorization_code` with PKCE against the configured OIDC
provider. The redirect URI is **always** `https://<vays-host>/oauth2-redirect`
and must be registered with the provider for the configured `clientID`.
There is no built-in support for username/password flows.

The id-token is renewed in the background with the refresh token. If a
renewal fails for a passing reason (network, provider unreachable), requests
fail with a *Network Error* and the user stays signed in; only when the
provider rejects the refresh token (or issued none) the user is signed out
at the token's expiry and the stored tokens are removed. A login in progress
belongs to its browser tab, so several tabs can sign in at the same time.

## Backend Outages

When a YAC backend cannot use its git repository, VAYS shows a banner at
the top of every page of that backend instead of error toasts:

  - **Maintenance / unavailable:** YAC answers `503`; the banner shows
    YAC's own title and message (e.g. *Data Repository in Maintenance*).
    Reads that fail this way show no data, writes and actions are not
    performed.
  - **Stale data:** YAC still answers reads from the last known state and
    flags them (`Warning: 110 ...` header); the banner tells when the data
    was last synced and that changes cannot be saved for now.

The banner disappears with the first normal response from that backend.
Pages that do not belong to a backend (login, help) never show it.
