---
layout: home
has_toc: false
---

![](assets/images/yac.png)
{: .text-center }

# About

## YAC

**YAC** (**Y**et **A**nother **C**onfigurator) manages many similar YAML files
in a GIT repository, but adds a layer of access control (inside the YAML data
structure) and allows to hook actions into the process.

It is a RESTful cloud-native web service that is configured through a Jinja2-
extensible YAML file. It uses OpenID Connect for user authentication, and an
extended JSON-Schema syntax to specify the YAML data stuctures and user access
control.

## VAYS

**VAYS** (**V**isual **A**dminInterface for **Y**AC **S**ervers) is the
official web frontend for one or more **YAC** backends. It is a single-page
React application served as a static bundle; all of its logic runs
in the user's browser, talking to YAC's REST API directly.

A single VAYS instance can connect to multiple YAC backends at once
(*backends* in the config) and lets users:

  - browse, filter and edit entities listed by each YAC instance,
  - create / copy / link / rename / delete entities,
  - run actions and inspect logs,
  - authenticate via the same OpenID Connect provider as YAC.

Forms are rendered from the JSON-Schema and UI-Schema that YAC generates
from the [specs file](yac/specs/index.md).
