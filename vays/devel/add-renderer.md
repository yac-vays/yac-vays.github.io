---
parent: Development
grand_parent: VAYS
nav_order: 5
---

# Adding a Renderer

VAYS renderers are JSON-Forms renderer/tester pairs, collected in
`src/renderers/index.tsx` from the index files of its subdirectories. To add a new one:

  1. Create a `MyRenderer.tsx` exporting a renderer component plus a
     `MyRendererTester` built with
     `rankWith(rank, isCustomRenderer('my_name'))` (or any other
     tester combination). Choose a rank above 20 (the existing explicit
     renderers use 22-31), otherwise a Material fallback renderer may win.
  2. Add the renderer/tester pair to the relevant index file
     (`control/`, `combined/`, `layout/` or `control/special/`).
  3. Add the name to `EXPLICIT_RENDERERS` in
     `src/utils/schema/schemaLint.ts` (and, where applicable, to the
     `STRING_RENDERERS` / `ARRAY_RENDERERS` / `NUMBER_RENDERERS` /
     `RENDERERS_IGNORING_INITIAL` sets there), otherwise the Schema
     Warnings report it as an unknown renderer.
  4. Add a test for the tester in `tests/renderers/testers.test.ts` and a
     page under `vays/renderers/` of the docs.
  5. Reference it from the YAC spec via
     `vays_options.renderer: my_name`.

See the [renderer source tree](https://github.com/yac-vays/vays/tree/main/src/renderers)
and its [README](https://github.com/yac-vays/vays/blob/main/src/renderers/README.md)
for conventions and helper utilities (`isCustomRenderer`,
`isUntypedStringInput`, ...). The
[Renderers documentation](../renderers/index.md) describes the bundled
renderers and how selection works at runtime. The
[JSON Forms primer](json-forms.md) covers the renderer/tester model
itself.
