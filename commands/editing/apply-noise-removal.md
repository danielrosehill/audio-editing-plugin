---
description: Run DeepFilterNet on an audio file and write a denoised version with _denoised suffix
argument-hint: <input-file> [--attenuation-limit 100]
---

Apply DeepFilterNet to `$1` and produce `<name>_denoised.<ext>` in the same directory.

## Steps

1. Read `${CLAUDE_PLUGIN_ROOT}/data/tools.json`. If `deep_filter.installed` is false, tell the user to run `/install-noise-removal` first and stop.
2. Resolve output path (same dir, stem + `_denoised`, same extension). Refuse to overwrite unless `--force`.
3. DeepFilterNet works on 48 kHz WAV. Build a pipeline:
   ```
   ffmpeg -i "$INPUT" -ar 48000 -ac 1 -c:a pcm_s16le "$TMPWAV"
   deep-filter --atten-lim <attenuation-limit> -o "$TMPOUT_DIR" "$TMPWAV"
   ```
4. Re-encode the denoised WAV back to match the input's original codec, bitrate, sample rate, and channel count (use `ffprobe` on the original to read these). Write to `$OUTPUT`.
5. Clean up tmp files.
6. Report input/output size, codec, and path.

## Defaults

- `attenuation-limit`: 100 (dB, i.e., no limit — most aggressive denoising). Lower this to 20–30 for a gentler touch that preserves room ambience.

Use `/prep-for-transcription` if the goal is a transcription-ready file rather than a clean listening copy.
