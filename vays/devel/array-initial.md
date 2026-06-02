---
parent: Development
grand_parent: VAYS
nav_order: 4
---

# Design Memo: `initial` on Arrays

A design memo, not a feature description — captured so the next
person to look at this doesn't have to rediscover the trade-offs.

## The Problem

The array renderer doesn't itself own the data. It recurses into
child renderers and only handles "add item" / "remove item" callbacks
on the outside. `initial` works the other way around: editing one
element has consequences for *every* element, even though the renderer
doing the edit can't see its siblings.

That mismatch is what makes `initial` on arrays awkward.

## Expected Behaviour

In an ideal world both `initial_editable: true` and
`initial_editable: false` work for arrays. The second case is
arguably not worth supporting; this memo is about the first.

### `initial_editable: true`

Editing one item materialises every item into `data`.

The catch: child renderers probably can't write the data themselves.
While the list isn't in `data` yet, the data path (the `path` in
`ControlProps`) is invalid.

### `initial_editable: false`

Questionable from a UX standpoint — editing one item makes all the
others disappear. Probably not worth the implementation effort.

## What Has to Change

Two threads run through this: the **data** side (writing the array
down) and the **UI-Schema** side (telling child renderers how to
behave). And there's a recursive case to keep in mind — you can have
a list of objects whose properties are themselves lists.

The shape of a fix:

  1. **Inject `initial` into the recursive UI-Schema call.** Without
     this, the child has no data entry to read from.
  2. **Intercept the child's writes at the array renderer.** Once the
     user edits one element, every element needs to write itself
     down. JSON Forms doesn't really support sibling-renderer
     communication, so the array renderer has to set the data itself
     and stop the children from doing it. The least bad signalling
     channel for that is the UI-Schema — flagging the child's
     control/complex wrapper to skip the write.

## Alternatives Worth Exploring

  - Step 2 might disappear entirely if we rewrite the
    `control`/`complexwrapper` HOCs that get applied at
    `export default` time. That avoids the UI-Schema hack at the
    cost of a deeper change.
  - An array-control renderer that accepts string items only.
    Drastically simpler — drastically less general.
