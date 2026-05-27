---
parent: VAYS
nav_order: 5
---

# Development

VAYS is a [React](https://react.dev) + [TypeScript](https://www.typescriptlang.org)
single-page application built with [Vite](https://vite.dev) and
[JSON Forms](https://jsonforms.io).

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
serves.

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

See [Installation](install.md) for full deployment options.

## Renderers

If you want to add a new form renderer, see the
[Renderers / Adding a Custom Renderer](renderers/custom.md) page and the
[`src/renderers/README.md`](https://github.com/yac-vays/vays/blob/main/src/renderers/README.md)
in the source tree for conventions.

## Releasing

`scripts/release.sh` calculates the next version tag from the current
branch (`testing` for `rc`, `main` for stable) and pushes it, which
triggers the CI build & deploy pipeline:

```sh
./scripts/release.sh minor   # or: major
```

  - On the `testing` branch a release-candidate tag (`vX.YrcN`) is
    created and the resulting image is published as `testing` and
    `vX.YrcN` on Docker Hub.
  - On the `main` branch a stable tag (`vX.Y`) is created and the image
    is published as `latest`, `vX`, `vX.Y`.

The script refuses to run with uncommitted changes or on any other
branch; see [Installation](install.md#versioning--container-tags) for
the full tag schema.

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

For reference (also see [Configuration](config.md#url-schema)):

```
domain/{backend-name}/{type}/?{filter-key}={filter-value}#{Page}.{NumPerPage}
domain/{backend-name}/{type}/create/{name}?view=...
```
