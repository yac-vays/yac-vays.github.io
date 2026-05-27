---
parent: Renderers
grand_parent: VAYS
nav_order: 4
---

# Adding a Custom Renderer

VAYS renderers are JSON-Forms renderer/tester pairs registered in
`src/renderers/index.tsx`. To add a new one:

  1. Create a `MyRenderer.tsx` exporting a renderer component plus a
     `MyRendererTester` built with `rankWith(rank, isCustomRenderer('my_name'))`.
  2. Add it to the relevant index file (`control/`, `combined/`,
     `layout/` or `control/special/`).
  3. Reference it from the YAC spec via
     `vays_options.renderer: my_name`.

See the [renderer source tree](https://github.com/yac-vays/vays/tree/main/src/renderers)
and its [README](https://github.com/yac-vays/vays/blob/main/src/renderers/README.md)
for conventions and helper utilities (`isCustomRenderer`,
`isUntypedStringInput`, ...).
