---
parent: Usage
nav_order: 3
---

# Custom Renderers
Custom renderers allow to render data for which a strict format is expected.


For VAYS to know when to apply a custom renderer is typically done using pattern matching on the json schema and/or the UI schema.
In most cases, the structure of the json schema itself is enough data to decide what renderer needs to be picked when. In some cases however, the YAC admin explicitly needs to specify what type of renderer needs to be used. This may be needed e.g. because by default a different renderer would be used to render the parameter, for example.



## Implicit Custom Renderers

Implicit custom renderers inspect the logic inside the json schema to decide whether they may be applied. You do not need to do anything special to enable them - you use them implicitly by the structures you define in the YAC config.


## Explicit Custom Renderers

There are a number of renderers that need to be named explicitly in the YAC configuration to make sure that they are used.


In the YAC config, one specifies the renderer like this:

```yaml
some_key:
    type: ...

    vays_options:
        renderer: <ENTER EXPLICIT RENDERER HERE>
        <RENDERER OPTION (if applicable)>: ...
```


Renderer Options:
- `password`:           A password renderer that only sends the Unix Crypt-SHA512 hash. (This means, the password hash looks
                        like in /etc/shadow
                        with the algorithm chosen is SHA-512, that is ID 6).

    Parameters:
                        
    `save_password_as`: Currently either `plaintext` or `crypt-sha-512`. (defaults to `crypt-sha-512`)

- `info_box`:           This renderer will display an info box where only `title` and `description` are relevant.

- `list_as_string`:     A renderer that will take an ansible-side string which is written as an enumeration (like "a,b,c") 
                        but it will display it to the user as a list. You can specify the corresponding separator as a renderer option.

    Parameters:

    `separator`:        string

- `big_string_list`:    A long list of strings. Make sure that the type is indeed strings (number may work, 
                        has not been tested yet).

- `multi_checkbox`:     Make sure that the type is `array`, using `oneOf`.
