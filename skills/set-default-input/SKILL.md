---
name: set-default-input
description: Set the default Pipewire audio input (source / microphone) right now. Use for one-off mic switches; for "always prefer this mic when connected" use microphone-preference-rules instead.
---

# Set Default Input

Switch the current default microphone.

## Steps

1. Resolve the ID via `list-audio-devices` if the user named the device by description.
2. Set:
   ```bash
   wpctl set-default <source-id>
   ```
3. Confirm with `wpctl status` — the chosen source should be marked default.

## Quick level sanity check

After switching, verify the new mic actually picks up audio — call `mic-level-check` for a few seconds of RMS readings, or run:

```bash
wpctl set-volume <source-id> 1.0
parecord --device=@DEFAULT_SOURCE@ --record-time=2 /tmp/miccheck.wav 2>/dev/null
```

## When to escalate to a persistent rule

If the user is repeatedly running this skill for the same device ("always pick the Jabra when it's there"), suggest moving to `microphone-preference-rules` so Wireplumber handles it automatically on connect.
