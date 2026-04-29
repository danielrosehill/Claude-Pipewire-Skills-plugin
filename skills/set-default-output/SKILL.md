---
name: set-default-output
description: Set the default Pipewire audio output (sink) — speakers, headphones, HDMI, Bluetooth, etc. Use when the user wants to switch where system audio plays right now, without writing a persistent rule.
---

# Set Default Output

Change the current default sink (audio output). For *persistent* "always prefer X when connected" behaviour, use `microphone-preference-rules` (analogous skill exists for outputs — extend that pattern).

## Steps

1. If the user named a device by description (e.g. "the Jabra"), call `list-audio-devices` first to resolve the ID.
2. Set the default:
   ```bash
   wpctl set-default <sink-id>
   ```
3. Verify:
   ```bash
   wpctl status | head -40
   ```
   The chosen sink should now have `*` next to it.

## Moving currently-playing streams

Setting the default sink does **not** move already-running streams by default on every system. To move them too:

```bash
for stream in $(wpctl status | awk '/Streams:/{f=1;next} f && /│.*├/ {print $2}' | tr -d '.'); do
  wpctl move "$stream" <sink-id>
done
```

A more reliable approach is `pw-cli` with `move-node`, or use `pactl move-sink-input <stream-id> <sink-name>` for each PulseAudio-style stream.

## When the user asks for "output also goes to X"

That's not a default change — that's a combined/loopback sink. Defer to the `route-on-device-connect` skill, which can set up a `module-combine-sink` style routing so audio plays to two outputs simultaneously.
