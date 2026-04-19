---
description: Denoise audio using ffmpeg's built-in RNNoise (arnndn) — fast, CPU-only, no external install
argument-hint: <input-file> [--model somnambulant|beguiling|leavened|conjoined-burgers|marathon|general]
---

Apply RNNoise via ffmpeg's `arnndn` filter and write `<name>_rnnoise.<ext>`.

## Steps

1. Ensure a model file exists at `~/.local/share/audio-editing/rnnoise/<model>.rnnn`. If missing, download from `https://github.com/GregorR/rnnoise-models`:
   ```
   mkdir -p ~/.local/share/audio-editing/rnnoise
   cd ~/.local/share/audio-editing/rnnoise
   curl -L -O https://raw.githubusercontent.com/GregorR/rnnoise-models/master/somnambulant-42000/sh.rnnn
   mv sh.rnnn somnambulant.rnnn
   ```
   Default model: `somnambulant` (good all-rounder for speech).
2. Run:
   ```
   ffmpeg -hide_banner -loglevel warning -i "$INPUT" \
     -af "arnndn=m=$MODEL_PATH" \
     -c:a <match-input-codec> -b:a <match-input-bitrate> \
     "$OUTPUT"
   ```
3. Output naming: same dir, stem + `_rnnoise` + same extension. Refuse overwrite unless `--force`.
4. Report input/output LUFS, path, and which model was used.

## Model hints

- `somnambulant` — balanced, best default for podcast/interview
- `marathon` / `leavened` — more aggressive, may dull sibilants
- `beguiling` — gentler, preserves breath and room

If DeepFilterNet is installed, prefer `/apply-noise-removal` for higher quality at cost of more CPU.
