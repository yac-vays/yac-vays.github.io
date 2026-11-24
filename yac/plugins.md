---
parent: YAC
nav_order: 4
---

# Plugins

To install a plugin, it is enough to copy/mount a file into the correct
location in `/code/app/plugin/{type}` of the YAC container. Each plugin type
is a bit different in its handling, see the according `README.md` for details
and the existing plugins source code as examples:

  - Type [action](https://github.com/yac-vays/yac/tree/main/app/plugin/action)
  - Type [format](https://github.com/yac-vays/yac/tree/main/app/plugin/format)
  - Type [j2_filters](https://github.com/yac-vays/yac/tree/main/app/plugin/j2_filters)
  - Type [j2_functions](https://github.com/yac-vays/yac/tree/main/app/plugin/j2_functions)
  - Type [j2_tests](https://github.com/yac-vays/yac/tree/main/app/plugin/j2_tests)
  - Type [json_schema](https://github.com/yac-vays/yac/tree/main/app/plugin/json_schema)
  - Type [log](https://github.com/yac-vays/yac/tree/main/app/plugin/log)
  - Type [repo](https://github.com/yac-vays/yac/tree/main/app/plugin/repo)
  - Type [schema_formats](https://github.com/yac-vays/yac/tree/main/app/plugin/schema_formats)
  - Type [ui_schema](https://github.com/yac-vays/yac/tree/main/app/plugin/ui_schema)
  - Type [validator](https://github.com/yac-vays/yac/tree/main/app/plugin/validator)

The plugin files of a type are loaded in alphabetical order.

For the function-style types (`j2_filters`, `j2_functions`, `j2_tests`,
`schema_formats`), every public function **defined in the plugin file** becomes
available under its name. Imported functions and those starting with `_` do
not, so helpers can be used freely.

The class-style types implement an abstract interface of `app.model.plg`; a
plugin class that lacks one of its methods fails when it is loaded.
`json_schema`, `ui_schema` and `validator` plugins run in the order their
`order()` returns. For the builtin `json_schema` plugins this is: `omit`
(so {% raw %}`{{ omit }}`{% endraw %} works for every keyword below), `yac_types`,
`yac_perms`, `yac_editable`, `yac_if`, `format`.
