---
name: list-audio-devices
description: List Pipewire audio sinks and sources with their IDs, descriptions, default-status, and current state. Use when the user asks "what mics/speakers are connected", needs IDs to feed into other skills, or is debugging why an expected device is missing.
---

# List Audio Devices

Enumerate every Pipewire sink (output) and source (input) currently visible to Wireplumber, plus which one is the default.

## Primary commands

Prefer `wpctl` (Wireplumber CLI) — cleaner output and aligns with how preference rules are written.

```bash
wpctl status
```

For machine-parsable output:

```bash
wpctl status --json 2>/dev/null || pw-dump
```

## What to surface to the user

For each sink/source:
- ID (the numeric ID `wpctl` uses)
- `node.name` (stable identifier — what rules match against)
- `node.description` (human-readable)
- Whether it is the current default (marked with `*` in `wpctl status`)
- Mute / volume state if relevant to the question

## Detecting `node.name` for rule writing

When the user wants a rule based on a device, get the canonical `node.name` (this is what Wireplumber matches on, not the description):

```bash
wpctl inspect <id> | grep -E 'node\.(name|description)|device\.product\.name'
```

## Notes

- Bluetooth devices appear/disappear with connection state — if a device the user expects is missing, suggest checking `bluetoothctl devices Connected`.
- USB device IDs change across reconnects; `node.name` is stable enough for rules in most cases. For absolute stability, match on `device.product.name` or `node.nick`.
