---
name: easy-effects-bindings
description: STUB — manage EasyEffects presets and bindings. Apply/remove input or output presets (compressor, EQ, RNNoise GUI, etc.) and bind a preset to a specific device so it auto-loads on connect.
---

# EasyEffects Bindings (Stub)

This skill is a placeholder — flesh out as the user's EasyEffects workflow stabilises.

## Intended scope

- List installed presets: `~/.config/easyeffects/{input,output}/`.
- Apply a preset via the EasyEffects D-Bus interface or CLI:
  ```bash
  easyeffects --load-preset <name>
  ```
- Bind a preset to a specific device so EasyEffects auto-applies it on connect — configured inside the EasyEffects GUI under each preset's device-binding tab. Backing storage is JSON in the preset file; this skill should be able to set the binding without opening the GUI.
- Toggle EasyEffects on/off:
  ```bash
  systemctl --user start easyeffects.service
  systemctl --user stop  easyeffects.service
  ```

## Open questions before implementing

- Does the user want EasyEffects always running, or launched on-demand for dictation only?
- For overlap with `dictation-mic-tune`: EasyEffects can host RNNoise as a node — should this skill replace the filter-chain approach in `dictation-mic-tune`, or sit alongside it as a heavier alternative?
- Preset library — does the user keep presets in version control (a separate repo)? If yes, this skill should reference the source-of-truth path instead of editing in-place.

Mark `TODO` and revisit once usage patterns are clearer.
