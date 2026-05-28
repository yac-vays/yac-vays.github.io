---
parent: Renderers
grand_parent: VAYS
nav_order: 14
---

# Renderer `mac_address`

A text input that knows about IEEE 802 MAC addresses. It auto-formats
what the user types into the canonical `AA:BB:CC:DD:EE:FF` shape and
surfaces an inline error while the value isn't yet a valid MAC.

## Behaviour

  - **Canonicalisation.** Hyphens (`-`) are rewritten to colons (`:`),
    letters are upper-cased, and any character that isn't a hex digit
    or `:` is stripped. The user can paste `aa-bb-cc-dd-ee-ff` or
    `aa.bb.cc.dd.ee.ff` and end up with `AA:BB:CC:DD:EE:FF` in the
    form.
  - **Live validation.** The displayed text is matched against
    `^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$`. Until it matches, an
    "Invalid MAC address format" hint appears under the field and the
    form data is set to `undefined` (so YAC sees the field as missing
    rather than as the partial value).

## Configuration

Select the renderer with `vays_options.renderer: mac_address`. The
renderer enforces the canonical MAC-address shape itself
(`^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$`) and clears the form value
whenever it doesn't match.

| Keyword     | Effect |
|:------------|:-------|
| `type`      | Must be `string` (or omitted) for the tester to match. |
| `pattern`   | Recommended: mirror the renderer's MAC regex so an empty value saved before the user types a valid address is also rejected by YAC. |

## [Specs](../../yac/specs/index.md) Example

{% raw %}
```yaml
schema:
  type: object
  properties:

    nic_mac:
      title: MAC Address
      description: The MAC address of the primary network interface.
      vays_category: Network
      type: string
      pattern: "^([0-9A-Fa-f]{2}:){5}[0-9A-Fa-f]{2}$"
      vays_options:
        renderer: mac_address
```
{% endraw %}
