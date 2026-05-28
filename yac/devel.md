---
parent: YAC
nav_order: 5
---

# Development

## Test & Build locally

To run the test suite and build locally, run the following command from within
the git repository:

```bash
sudo docker build --progress plain --build-arg version=v0.0 -t yac .
```

## Release

Commit and push your changes to the `test` branch if you want to make
a new **release-candidate**. Then run `scripts/release.sh` to tag your
commit with a new rc-version and start the
[build-pipeline](https://github.com/yac-vays/yac/blob/main/.gitlab-ci.yml).

To **release** a release-candidate, merge your commit to the `main` branch
and run `scripts/release.sh` from there (it will again tag your commit
with a new minor-version and start the build-pipeline).

## Upgrade Environment

- Check on [DockerHub](https://hub.docker.com/_/python) for new versions and
  adjust the tag in the `FROM` instruction of `./Dockerfile`. (Use a most
  specific tag to allow reproducable builds.)

- Build container (and update the requirements file) with:

      sudo docker build --progress plain -t yac .
      sudo docker run --rm -v "$(pwd)/requirements.in:/r.in:ro" --entrypoint sh yac:latest -c \
          "pip install pip-tools &>/dev/null; /home/yac/.local/bin/pip-compile -o - /r.in" > ./requirements.txt
