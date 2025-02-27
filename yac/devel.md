---
parent: YAC
nav_order: 5
---

# Development

{: .warning}
TODO explain common devel tasks

### Upgrade Environment

- Check on https://hub.docker.com/_/python for new versions and adjust the tag
  in the `FROM` instruction of `./Dockerfile`. (Use a most specific tag to allow
  reproducable builds.)

- Build container and update the requirements file with:

      docker run --rm -v "$(pwd)/requirements.in:/r.in:ro" --entrypoint sh yac:latest -c \
          "pip install pip-tools &>/dev/null; pip-compile -o - /r.in" > ./requirements.txt

      docker build --progress plain -t yac .
