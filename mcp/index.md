---
nav_order: 3
---

# MCP

The **MCP** server connects AI agents (Claude, ChatGPT, IDE assistants, ...) to
one or more YAC backends via the [Model Context
Protocol](https://modelcontextprotocol.io). The agent acts **as the logged-in
user**: YAC sees the user's own id-token, so permissions, roles and the commit
author are the same as in VAYS.

Like VAYS, it is a pure client of the YAC REST API. Agents work on the YAML
document instead of a form: read an entity, edit its YAML, validate it until
it is valid, then commit it.

## Connecting a Client

Add the server with its URL `https://<mcp-host>/mcp` (streamable HTTP) to the
MCP client. The client registers itself and opens a browser window for the
login at the OIDC provider (the same one as VAYS), followed by a consent page
of the MCP server. No tokens or passwords need to be copied.

## Tools

| Tool | Description |
|:-----|:------------|
| `list_types` | Backends and their entity types (with actions and logs). |
| `list_entities` | Entities of a type (with search). |
| `read_entity` | The YAML, the JSON schema, notes on special fields and an `edit_token`. |
| `new_entity` | A starting YAML for a new entity (with defaults). |
| `validate` | Checks a create or edit without committing (all schema errors at once). |
| `create`, `edit`, `delete` | Commit a change; `edit` takes the whole YAML or a `data` patch. |
| `run_action` | Runs an action on an entity. |
| `read_logs` | The logs of an entity, or the content document of one log. |

The write tools (`create`, `edit`, `delete`, `run_action`) are only offered if
the server [allows](install.md#allowed-changes) them (by default, it is
read-only).

As in VAYS, missing [defaults](../yac/specs/file/schema.md) are filled in, and
fields with the [renderers](../vays/renderers/index.md) `age_secret`,
`random_string` and `password` are handled by the server: values are
generated (or regenerated with `~generate`), passwords are passed as
cleartext in the separate `secrets` argument and hashed. Further renderers can
be added as plugins, see the
[README](https://github.com/yac-vays/mcp/blob/main/app/plugin/renderer/README.md).

## Guardrails

  - Every request to YAC carries the header `YAC-Client: mcp`. Declare it in
    the [request section](../yac/specs/file/request.md) of the specs (e.g. as
    `client` with the pattern `^mcp$`) to restrict agents in
    [roles](../yac/specs/file/roles.md) with `request.headers.client != 'mcp'`.
    This only limits a user's own agents: calling YAC directly bypasses it.
  - Commit messages end with `(via MCP)`.
  - There is no admin override (`force`), also not for admins.
  - Generated secrets (e.g. `age_secret` cleartexts) are not returned to the
    agent unless enabled.
  - Entity data of other users ends up in the agent's context, which makes
    prompt injection possible. Let the MCP client ask before write and delete
    tools (they are marked as such). The server itself does not ask: to keep
    agents from deleting or running actions at all, leave `delete` / `actions`
    out of the [allowed changes](install.md#allowed-changes).
