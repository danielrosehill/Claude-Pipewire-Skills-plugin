---
name: dictation-mic-tune
description: Diagnose and tune a microphone for speech-to-text dictation — evaluate level, apply RNNoise/noise-cancellation virtual source, set a higher software gain, and optionally persist the noise-cancelled source as the default. Use when the user dictates frequently and wants the mic to "just work" for STT.
---

# Dictation Mic Tune

End-to-end mic prep for dictation: measure → clean → boost → persist.

## Phase 1: Measure

Reuse the `mic-level-check` workflow first. Capture peak / RMS / clipping on the current source. Decide:

- Peak < -30 dB → needs gain boost.
- Peak > -3 dB → needs gain reduction (clipping ruins STT more than low gain does).
- High noise floor (RMS gap between speech and silence < 20 dB) → needs noise cancellation.

## Phase 2: Add a noise-cancelled virtual source

Pipewire ships `module-echo-cancel` and an RNNoise filter chain. RNNoise gives much better results for dictation than plain echo-cancel.

Create `~/.config/pipewire/pipewire.conf.d/52-rnnoise-source.conf`:

```
context.modules = [
  {   name = libpipewire-module-filter-chain
      args = {
          node.description = "Noise Cancelling source (RNNoise)"
          media.name       = "Noise Cancelling source"
          filter.graph = {
              nodes = [
                  {
                      type    = ladspa
                      name    = rnnoise
                      plugin  = /usr/lib/ladspa/librnnoise_ladspa.so
                      label   = noise_suppressor_mono
                      control = { "VAD Threshold (%)" = 50.0 }
                  }
              ]
          }
          capture.props = {
              node.name    = "capture.rnnoise_source"
              node.passive = true
              audio.rate   = 48000
          }
          playback.props = {
              node.name    = "rnnoise_source"
              media.class  = Audio/Source
              audio.rate   = 48000
          }
      }
  }
]
```

The plugin path varies — check both `/usr/lib/ladspa/` and `/usr/lib/x86_64-linux-gnu/ladspa/`. Install with `sudo apt install rnnoise-ladspa-plugin` on Debian/Ubuntu, or `noise-suppression-for-voice` from upstream.

Restart: `systemctl --user restart pipewire wireplumber`.

A new source `Noise Cancelling source (RNNoise)` will appear. Route the user's input through it (set as default via `set-default-input`, or reference it directly in the dictation app).

## Phase 3: Gain staging

Once on the cleaned source, re-measure with `mic-level-check`. Apply software gain:

```bash
wpctl set-volume <rnnoise-source-id> 1.3   # 130% — Pipewire allows up to ~1.5 without significant artefacts
```

For per-app preference (so dictation apps go through the cleaned source while everything else stays on the raw mic), drop a Wireplumber rule matching the dictation app's `application.name` and routing it to the RNNoise source.

## Phase 4: Persist

The `.conf` files above survive reboots. Verify after a reboot:

```bash
systemctl --user status pipewire wireplumber
wpctl status | grep -i rnnoise
```

If the cleaned source disappears after reboot, the LADSPA plugin path in the config is likely wrong — check the actual install location with `dpkg -L rnnoise-ladspa-plugin` (or equivalent) and update the `plugin` line.

## When to escalate to EasyEffects

If the user wants a GUI to live-tweak gain / EQ / compressor / multiple chained filters, the RNNoise filter-chain approach hits a ceiling — switch to EasyEffects. See `easy-effects-bindings`.

## Reporting back

After tuning, give the user:
- Before/after peak & RMS dB
- Which virtual source was created and its node name
- Whether it's now the default input
- A one-liner to undo: `rm ~/.config/pipewire/pipewire.conf.d/52-rnnoise-source.conf && systemctl --user restart pipewire`
