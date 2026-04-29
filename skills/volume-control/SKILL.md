---
name: volume-control
description: Get, set, mute, or unmute volume on any Pipewire sink or source. Supports software gain >100% (up to ~150% cleanly). Use for one-off volume changes; persistent device-specific volume defaults belong in a Wireplumber rule.
---

# Volume Control

Wraps `wpctl get-volume` / `set-volume` / `set-mute`.

## Operations

```bash
# Get
wpctl get-volume <id>

# Set absolute (0.0 – 1.5; 1.0 = 100%)
wpctl set-volume <id> 0.8

# Relative
wpctl set-volume <id> 5%+
wpctl set-volume <id> 5%-

# Mute toggle
wpctl set-mute <id> toggle

# Unmute
wpctl set-mute <id> 0
```

`<id>` can be `@DEFAULT_SINK@` / `@DEFAULT_SOURCE@` for the current default device.

## Software gain caveat

`wpctl set-volume <id> 1.3` is "130% software gain" — useful for quiet mics or quiet recordings, but introduces digital amplification artefacts beyond ~1.5. For headphones / line-out, prefer hardware volume; for USB devices, software gain is often the only control.

## Persisting

`wpctl set-volume` doesn't persist by default — Wireplumber will reset it on next start. To make a per-device default volume sticky, add it as a `node.props.audio.gain` override in a Wireplumber rule (see `microphone-preference-rules` for the rule shape).
