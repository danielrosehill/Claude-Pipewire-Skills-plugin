---
name: route-on-device-connect
description: Set up combined-sink / fan-out output routing — e.g. "when the Jabra is connected, also play audio through it without changing the default speakers". Use for the case the GUI can't handle: simultaneous output to multiple sinks, or conditional routing tied to device connect state.
---

# Route On Device Connect

Pipewire's GUI mixers (pavucontrol, plasma-pa) only let you pick one default output. For "play to A *and* B" or "when device X connects, also send audio there", you need a combined sink or a Wireplumber rule.

## Approach 1: Persistent combined sink

Create a Pipewire combined sink that fans out to multiple physical sinks. Define it in `~/.config/pipewire/pipewire.conf.d/51-combine-sink.conf`:

```
context.modules = [
  {   name = libpipewire-module-combine-stream
      args = {
          combine.mode = sink
          node.name    = combined_speakers_and_jabra
          node.description = "Speakers + Jabra"
          combine.props = {
              audio.position = [ FL FR ]
          }
          stream.props = {}
          stream.rules = [
              {
                  matches = [
                      { node.name = "alsa_output.pci-0000_00_1f.3.analog-stereo" }
                      { node.name = "~alsa_output.usb-Jabra.*" }
                  ]
                  actions = { create-stream = {} }
              }
          ]
      }
  }
]
```

Then `systemctl --user restart pipewire wireplumber` and select the combined sink as default with `wpctl set-default`.

## Approach 2: Wireplumber rule that re-routes on connect

Use Wireplumber Lua scripting (`~/.config/wireplumber/scripts/`) to react to a device appearing and call `wpctl set-default` or move existing streams. Heavier — only reach for this if the combined-sink approach can't model the desired behaviour.

## Steps

1. Clarify with the user: do they want **simultaneous** output (combined sink) or **automatic switch** on connect (rule + default change)?
2. Resolve `node.name` for each device with `list-audio-devices`.
3. Drop the config file (the combined-sink template above is a good default).
4. `systemctl --user restart pipewire wireplumber`.
5. Test: with the device connected, the new combined sink shows in `wpctl status`. Set it default. Disconnect the device and confirm Pipewire degrades gracefully (the combined sink will lose one stream but keep playing through the remaining one).

## Caveats

- Combined sinks introduce a small amount of latency — fine for music/system sounds, noticeable for live monitoring.
- Different sample rates between the two physical sinks force resampling. Pipewire handles this automatically but quality can dip. If both sinks support 48kHz, set them to that explicitly.
- For Bluetooth, A2DP-only profiles change `node.name` — the regex match in the example handles this; verify after connecting.
