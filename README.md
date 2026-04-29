# Claude Pipewire Skills

Claude Code skills for taming Pipewire / Wireplumber audio on Linux. Pipewire is powerful but its CLI surface is sprawling and the GUI mixers can't express the rules you actually want — like "always prefer this mic when it's connected" or "play to the speakers *and* the headset simultaneously". These skills fill that gap.

Aimed at users who use voice dictation heavily and want their mics, gain staging, and noise cancellation configured proactively rather than reactively.

## Skills

### Inspection & one-off control
- `list-audio-devices` — enumerate sinks and sources with their IDs and `node.name` keys.
- `set-default-output` — switch the current default sink.
- `set-default-input` — switch the current default source / microphone.
- `volume-control` — get / set / mute on any sink or source, including software gain >100%.

### Persistent rules
- `microphone-preference-rules` — write Wireplumber rules so a preferred mic is auto-selected on connect, with a fallback when it disappears.
- `list-preference-rules` — show every custom rule currently active and what it does.
- `delete-preference-rule` — remove a rule cleanly and restart Wireplumber.
- `route-on-device-connect` — combined-sink / fan-out routing for "play to A and B" cases the GUI can't handle.

### Dictation tuning
- `mic-level-check` — quick peak/RMS/clipping diagnostic on any source.
- `dictation-mic-tune` — measure → apply RNNoise filter chain → set gain → persist as a default-ready source.
- `easy-effects-bindings` — STUB for managing EasyEffects presets and per-device bindings.

## Installation

```bash
claude plugins install claude-pipewire-skills@danielrosehill
```

Or at project scope from inside a specific repo:

```bash
claude plugins install claude-pipewire-skills@danielrosehill --scope project
```

## Requirements

- Pipewire ≥ 0.4 with Wireplumber 0.5+ (default on Ubuntu 24.04 / Fedora 40 / Arch current).
- `wpctl`, `pw-cli`, `pw-dump` (ship with Pipewire).
- `parecord` and `sox` for level analysis.
- For `dictation-mic-tune`: an RNNoise LADSPA plugin (e.g. `rnnoise-ladspa-plugin` on Debian/Ubuntu, or `noise-suppression-for-voice` from upstream).

## License

MIT
