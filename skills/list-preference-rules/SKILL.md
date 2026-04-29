---
name: list-preference-rules
description: List all custom Wireplumber rules currently active on the system — device priorities, routing rules, profile overrides. Use to see what's been set before adding/removing.
---

# List Preference Rules

Show the user every custom Wireplumber rule file and a one-line summary of what each does.

## Where to look

```bash
ls -la ~/.config/wireplumber/wireplumber.conf.d/ 2>/dev/null
ls -la /etc/wireplumber/wireplumber.conf.d/ 2>/dev/null
```

User-level files override system-level ones with the same name.

## Summarise each file

For every `.conf` file found, parse out:
- The match key (`node.name`, `device.product.name`, etc.) and the matched value
- The action (priority change, prop update, disabled, etc.)
- The likely effect ("makes Jabra the preferred mic", "disables the laptop speakers")

Don't dump the raw file unless the user asks — give the human-readable summary first.

## Cross-check against current state

Compare each rule's matched device to the output of `wpctl status`. Flag rules whose target device is not currently present (orphaned rules — user may want to delete).
