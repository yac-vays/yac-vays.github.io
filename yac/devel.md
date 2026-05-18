---
parent: YAC
nav_order: 5
---

# Development

## Build & Upgrade

YAC is shipped as a container.

See [README.md](https://github.com/yac-vays/yac/blob/main/README.md) in the
repository for the most important development steps.

The Dockerfile has three stages: `build` (install requirements + copy app),
`test` (runs `pylint` and the unit tests in `tests/`) and `production` (the
final image). The `production` stage depends on `test`, so a failing test
will fail the build.

## Branches and Tags

The git repository has two branches, `test` and `main`. Commits will not
immediately trigger the build-pipeline. Only tags will. Push tags like
`v1.3rc7` (in incrementing order) to the `test` branch for test releases.
Then merge them into `main`, tagged as `v1.3` for production releases.

Development can always also happen in short-lived dev-branches and be merged
into test for test-releases.

Changes to the helm-chart will always be published as a new helm-chart
version (independent of branches or tags)!

## Plugins

To add or modify a plugin during development, drop the file into
`app/plugin/{type}/{name}.py` and restart YAC. See [Plugins](plugins.md) and
the `README.md` next to each plugin type.
