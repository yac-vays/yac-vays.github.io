---
parent: Development
grand_parent: VAYS
nav_order: 6
---

# Adding a Log Renderer

[Log renderers](../log_renderers/index.md) live in
`src/view/components/EntityList/Logs/renderers/` and are registered by name in
its `index.ts`; the name is what spec authors put into `renderer` of a log. A
renderer implements `LogRenderer` (`types.ts`):

| Member | Description |
|:-------|:------------|
| `name` | The name used in the specs. |
| `Indicator` | Component filling the symbol's square in the status column, gets the log's declaration (`decl`, incl. `options`) and its newest entry (`latest`). |
| `display` | `popover` (the view opens next to the symbol, for short content) or `window` (a large dialog). |
| `View` | Component for the details, gets `decl`, all `entries` (newest first), `entityName`, `requestContext` and `onClose`. |

`LogsField` does everything common to all renderers: polling the entries, the
square of each symbol (in the look of the action buttons, grouped, with the
newest message as tooltip), the loading spinner and the dash of a log without
entries, opening and closing the view.

To add one:

  1. Create `myrenderer.tsx` exporting a `LogRenderer`. Reuse what fits:
      - the building blocks in `symbols.tsx` for the square's content:
        `SymbolIcon` (a material symbol in a colour), `SymbolText` (e.g. a
        percentage), `SymbolBar` (a bar along the bottom edge),
        `SymbolWarning` (the amber corner sign) and the colours `GREEN`,
        `RED`, `AMBER`, `GREY`,
      - `LogPanel` (one directory up, in `Logs/`) for a list of entries in a
        popover,
      - `LogWindow` (in the look of the VAYS dialogs: header with title,
        subtitle, your round `WindowButton`s and close; closes on `Esc` and on
        the backdrop) for a window,
      - `useLogContent` + `LogContentStatus` to load the log's content
        document when the view opens (with loading / error / retry).
  2. Register it in `LOG_RENDERERS` in `index.ts`.
  3. Add tests (see `tests/view/logRenderers.test.tsx`) and a page in
     `vays/log_renderers/` of these docs.

Keep the indicator cheap: it is rendered for every row and re-rendered on each
poll. Anything big (documents) belongs into the view, fetched on open — YAC's
`GET /entity/{type}/{name}/logs/{log}/content`, which the log plugin provides
via `content()`.
