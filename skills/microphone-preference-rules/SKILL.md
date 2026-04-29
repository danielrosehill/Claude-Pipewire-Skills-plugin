---
name: microphone-preference-rules
description: Create persistent Wireplumber rules that auto-prefer specific microphones when connected — e.g. "if the Jabra headset is connected, always make it the default mic; otherwise fall back to the laptop array". Use when the user is tired of manually switching mics.
---

# Microphone Preference Rules

Write persistent device-priority rules so Wireplumber automatically picks the right mic without manual switching after every reconnect.

## Where rules live

Wireplumber Lua/Spa-JSON config (Pipewire ≥ 0.4.x with Wireplumber 0.5+):

```
~/.config/wireplumber/wireplumber.conf.d/
```

Use a single file per logical rule set (e.g. `51-mic-priority.conf`) so rules are easy to find and remove.

## Rule shape (Spa-JSON)

```
monitor.alsa.rules = [
  {
    matches = [
      {
        node.name = "alsa_input.usb-Jabra_Jabra_Evolve2_85-00.mono-fallback"
      }
    ]
    actions = {
      update-props = {
        priority.session = 2000
        priority.driver  = 2000
      }
    }
  }
  {
    matches = [
      {
        node.name = "alsa_input.pci-0000_00_1f.3.analog-stereo"
      }
    ]
    actions = {
      update-props = {
        priority.session = 100
      }
    }
  }
]
```

Higher `priority.session` wins. Wireplumber re-evaluates default device on connect/disconnect.

## Steps

1. Get the canonical `node.name` of each mic the user wants to rank — call `list-audio-devices` and `wpctl inspect <id>`.
2. Ask the user for the desired ordering (preferred → fallback).
3. Drop the file under `~/.config/wireplumber/wireplumber.conf.d/`. Use a numeric prefix (e.g. `51-`) so it loads after the defaults.
4. Restart Wireplumber for the user-only session:
   ```bash
   systemctl --user restart wireplumber
   ```
5. Verify with `wpctl status` — the preferred mic should now be marked default when connected; disconnect it and confirm the fallback takes over.

## Matching tips

- `node.name` is the most stable single key — but if a USB hub renames the device on reconnect, fall back to matching on `device.product.name` or `device.serial`.
- For Bluetooth, the node name often includes the MAC address, which is stable. Use `wpctl inspect` to confirm.
- Wildcards: use `matches.*.node.name = "~alsa_input.usb-Jabra.*"` for a regex match.

## Bluetooth caveat

Bluetooth headsets switch between A2DP (high-quality output, no mic) and HSP/HFP (mic + low-quality output). The mic only exists in HSP/HFP profile — make sure the rule matches the HSP/HFP `node.name`, not the A2DP one.

## Companion skills

- `list-preference-rules` — show what rules are currently active.
- `delete-preference-rule` — remove a rule by file or by matched device.
