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
[Architecture](arch.md) overview, then skim the
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

## Releasing

Commit and push your changes to the `test` branch if you want to make
a new **release-candidate**. Then run `scripts/release.sh` to tag your
commit with a new rc-version and start the build-pipeline.

To **release** a release-candidate, merge your commit to the `main` branch
and run `scripts/release.sh` from there (it will again tag your commit
with a new minor-version and start the build-pipeline).

## Upgrade Environment

- Check on DockerHub for new versions of [Node](https://hub.docker.com/_/node) and
  [Nginx](https://hub.docker.com/r/nginxinc/nginx-unprivileged) and
  adjust the tags in the `FROM` instructions of `./Dockerfile`. (Use a most
  specific tag to allow reproducable builds.)

- Update npm dependencies (`package.json` and `package-lock.json`) with the
  following commands (run them separately - combining them can lead to
  inconsistent updates):

  ```bash
  npm update --save
  npm update --save-dev
  ```

## URL Schema

For reference (also see [Configuration](../config.md#url-schema)). The full
route list (see `src/App.tsx`):

```
/                                       Login (OIDC)
/oauth2-redirect                        OAuth2 callback
/{backend-name}/{type}/                 Overview (list of entities)
/{backend-name}/{type}/create/{name}?   Create (or copy) an entity
/{backend-name}/{type}/edit/{name}?   Edit an entity
/{backend-name}/{type}/read/{name}?     Read an entity (read-only)
/error-page                             Error page
/dev-info                               Developer info
```

The overview route additionally supports filtering and paging via query
string and fragment:

```
domain/{backend-name}/{type}/?{filter-key}={filter-value}#{Page}.{NumPerPage}
```
