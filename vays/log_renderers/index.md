---
parent: VAYS
nav_order: 4
has_children: true
---

# Log Renderers

The entity list shows the [logs](../../yac/specs/file/types/logs.md) of each
entity in its *Status* column: one symbol per log, which opens the log's
details on click. How a log is shown is decided by its *log renderer*,
selected with `renderer` on the log in the YAC specs (default `messages`).
Renderer-specific settings go into `options` of the log.

The renderer is independent of where YAC gets the log from (its `plugin`):
every renderer works with every plugin, as long as the plugin provides what
the renderer uses (e.g. a `progress` for `progress`, a content document for
`document`).

## Common Behaviour

  - **Symbol.** Each renderer builds its symbol from the **newest entry** of the
    log. While the logs are loading, and for a log without entries, all
    renderers show the same grey placeholder (a spinner / a question mark).
  - **Polling.** The entries are refreshed in the background every few
    seconds while the list is shown (and right after an operation on the
    entity). Content documents are not polled: they are fetched each time they
    are opened.
  - **Entry fields.** `problem` and `progress` of the entries are generic: the
    list of entries marks problems with a red `!` and shows a progress column
    if an entry reports a progress, whatever the renderer.
  - **Popover or window.** The details open either in a popover next to the
    symbol (short content) or in a large window (documents). A window closes
    with its close button, `Esc` or a click next to it.

## Renderers

| Renderer                    | Symbol                                           | Opens                                   | Since |
|:----------------------------|:-------------------------------------------------|:----------------------------------------|------:|
| [`messages`](messages.md)   | Neutral "info" symbol.                           | Popover with the entries.               |    v0 |
| [`problem`](problem.md)     | Green check / red exclamation mark from `problem`. | Popover with the entries.               |    v0 |
| [`progress`](progress.md)   | Progress ring from `progress` (exclamation mark on `problem`). | Popover with the entries and their progress. |    v0 |
| [`document`](document.md)   | Document symbol, light warning on `problem`.     | Window with the content document (highlighting, search, copy, download). |    v0 |
| [`markdown`](markdown.md)   | Text symbol, light warning on `problem`.         | Window with the content document rendered as Markdown. |    v0 |

An unknown `renderer` is shown as `messages` (and logged in the browser
console). To add a renderer, see
[Adding a Log Renderer](../devel/add-log-renderer.md).
