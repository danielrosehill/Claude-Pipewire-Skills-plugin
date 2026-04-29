---
name: mic-level-check
description: Sample a microphone for a few seconds and report peak/RMS level, clipping, and silence. Use to confirm a mic is actually picking up audio and roughly at the right gain. Quick diagnostic — for full dictation tuning use dictation-mic-tune.
---

# Mic Level Check

Capture a short clip from a source and report the level so the user can tell whether the mic is alive, muted, too quiet, or clipping.

## Steps

1. Resolve target source: default with `@DEFAULT_SOURCE@` unless the user specified a device (then resolve the ID/name via `list-audio-devices`).
2. Confirm it's not muted:
   ```bash
   wpctl get-volume <source-id>
   ```
3. Record ~3 seconds:
   ```bash
   parecord --device=@DEFAULT_SOURCE@ --record-time=3 --file-format=wav /tmp/miccheck.wav 2>/dev/null
   ```
   (Or `parec | sox` if `parecord` isn't available.)
4. Analyse with `sox`:
   ```bash
   sox /tmp/miccheck.wav -n stats 2>&1
   ```
   Capture: `RMS lev dB`, `Pk lev dB`, `Min/Max amplitude`.

## What to report

- **Peak dB**: ideal for speech is around -6 dB to -12 dB. Above -3 dB → risk of clipping. Below -30 dB → too quiet, raise gain.
- **RMS dB**: ideal -18 to -24 dB for speech.
- **Silence detected**: if peak is below -50 dB, mic is probably muted, broken, or has wrong device routing.
- **Clipping**: if `Max amplitude` ≥ 0.99, lower gain.

## Suggested fixes (offer, don't auto-apply)

- Too quiet → raise volume with `wpctl set-volume <source-id> 1.2` (Pipewire allows >1.0 software gain up to ~1.5 cleanly).
- Clipping → reduce volume; also check whether a hardware boost is enabled in `alsamixer`.
- Silent → check mute state, check `pavucontrol` Recording tab for app routing, check whether the right Bluetooth profile (HSP/HFP) is active.

For a richer dictation-tuned workflow (RNNoise, gain staging, persisted source), use `dictation-mic-tune`.
