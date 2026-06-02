---
parent: VAYS
nav_order: 5
has_children: true
---

# Development

VAYS is a [React](https://react.dev) + [TypeScript](https://www.typescriptlang.org)
single-page application built with [Vite](https://vite.dev) and
[JSON Forms](https://jsonforms.io).

New to the codebase? Start with the
[Architecture](architecture.md) overview, then skim the
[React](react.md) and [JSON Forms](json-forms.md) primers before
touching renderer code.

## Setup

Requires Node.js LTS (v22 — *Jod* — at the time of writing).

```sh
git clone https://github.com/yac-vays/vays.git
cd vays
npm ci
```

## Run Locally

The dev server uses HTTPS, so a local certificate is needed:

```sh
mkdir -p cert
openssl req -x509 -newkey rsa:3072 -nodes -sha256 \
    -subj '/CN=127.0.0.1' \
    -keyout cert/private-key.pem \
    -out  cert/certificate.pem
npm run dev
```

VAYS will be reachable at the URL printed by Vite. Note that most OIDC
providers won't accept the resulting self-signed origin as a redirect
URI; for end-to-end auth tests, use a Docker build behind a reverse
proxy with a real certificate.

## Build for Production

```sh
npm run clear   # only needed if `npm run dev` was used before
npm run build
```

The resulting bundle in `dist/` is what the production container
serves. It is a pure static bundle, so any web server can host it —
nginx, Caddy, S3 + CloudFront, ... Enable gzip (or brotli) compression
at the server to keep the over-the-wire size small (for nginx, that's
`gzip on;`). The bundled container image already does this; if you
self-host, you'll want to mirror the setting.

## Tests

```sh
npm run test    # vitest
npm run lint    # eslint
npm run pretty  # prettier
```

## Container Build

The same image used in production:

```sh
docker build --memory=2g -t vays .
docker run -p 8080:8080 \
    --mount type=bind,source="$(pwd)/config.json",target=/usr/share/nginx/html/config.json,readonly \
    vays
```

See [Installation](../install.md) for full deployment options.

## Adding a Renderer

VAYS renderers are JSON-Forms renderer/tester pairs registered in
`src/renderers/index.tsx`. To add a new one:

  1. Create a `MyRenderer.tsx` exporting a renderer component plus a
     `MyRendererTester` built with
     `rankWith(rank, isCustomRenderer('my_name'))` (or any other
     tester combination).
  2. Add the renderer/tester pair to the relevant index file
     (`control/`, `combined/`, `layout/` or `control/special/`).
  3. Reference it from the YAC spec via
     `vays_options.renderer: my_name`.

See the [renderer source tree](https://github.com/yac-vays/vays/tree/main/src/renderers)
and its [README](https://github.com/yac-vays/vays/blob/main/src/renderers/README.md)
for conventions and helper utilities (`isCustomRenderer`,
`isUntypedStringInput`, ...). The
[Renderers documentation](../renderers/) describes the bundled
renderers and how selection works at runtime. The
[JSON Forms primer](json-forms.md) covers the renderer/tester model
itself.

## Releasing

Commit and push your changes to the `test` branch if you want to make
a new **release-candidate**. Then run `scripts/release.sh` to tag your
commit with a new rc-version and start the build-pipeline.

To **release** a release-candidate, merge your commit to the `main` branch
and run `scripts/release.sh` from there (it will again tag your commit
with a new minor-version and start the build-pipeline).

## Upgrading Dependencies

```sh
npm update --save
npm update --save-dev
```

Run the two commands separately — combining the flags can lead to
inconsistent updates.

Periodically check [hub.docker.com/_/node](https://hub.docker.com/_/node)
and the unprivileged nginx base image for new versions and update the
tags in `Dockerfile`.

## URL Schema

For reference (also see [Configuration](../config.md#url-schema)):

```
domain/{backend-name}/{type}/?{filter-key}={filter-value}#{Page}.{NumPerPage}
domain/{backend-name}/{type}/create/{name}?view=...
```
