---
parent: VAYS
nav_order: 1
---

# Configuration

VAYS itself needs very little configuration. Most of it is done automatically on the
clientside using the data that is provided by YAC - and its configuration. Henc, it's own
configuration file (config.json) is relatively simple and typically does not change much over time.

Here is the basic skeleton:

```json
{
  "title": "The title of the VAYS instance",
  "color": {
    "primary" : "#00596D",
    "primaryHighlighted" : "#007894"
  },
  "production": "boolean, whether production mode is enabled",
  "oidcConf": {
    "server": "The access provider URL",
    "clientID": "The Audience"  
  },
  "backends": [
    {
      "name": "url_friendly_name",
      "title": "The Title in the GUI",
      "icon": "Some SVG",
      "url": "(The URL of the YAC backend)"
    }
  ],

  "logo": "The logo"
}
```


