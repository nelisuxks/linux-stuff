clean and simple `fastfetch` configuration, with custom formatting due to formatting of certain modules being not to my taste lol

```
{
  "$schema": "https://github.com/fastfetch-cli/fastfetch/raw/dev/doc/json_schema.json",
  "logo": {
      "type": "builtin",
        "height": 15,
        "width": 30,
        "padding": {
            "top": 2,
            "left": 3
        }
    },
  "modules": [

    "title",
    "separator",
    {
      "type": "kernel",
    },
    {
      "type": "os",
      "format": "{name} {version}"
    },
    "de",
    "wm",
    "terminal",
    "packages",
    "break",
    {
      "type": "host",
      "key": "Device",
      "format": "{vendor} {version}"
    },
    {
      "type": "cpu",
      "format": "your-cpu-name"
    },
    {
      "type": "gpu",
      "format": "your-gpu-name"
    },
    {
      "type": "display",
      "format": "your-display-specs"
    },
    "memory",
    "disk",
    "battery",
    "break",
    "uptime",
    "break",
    "colors"
  ]
}
```
>*Note:* in order to echo fastfetch output into the terminal at launch, `fastfetch` command needs to be added to the bottom of the `~/.bashrc` file

**example in kitty:**
