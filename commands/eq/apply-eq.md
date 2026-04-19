---
description: Apply a saved or built-in EQ preset to an audio file, writing <name>_eq-<preset>.<ext>
argument-hint: <preset-name> <input-file> [--force]
---

Apply EQ preset `$1` to file `$2`.

## Steps

1. Load `${CLAUDE_PLUGIN_ROOT}/data/eq-presets.json`. Look up `user_presets.$1` first, then `builtin_presets.$1`. Also check `~/.config/audio-editing/eq/$1.json`. If not found, list available presets and stop.
2. Resolve output: same dir as input, stem + `_eq-<preset>` + same extension. Refuse overwrite unless `--force`.
3. `ffprobe` the input to capture codec, bitrate, sample rate, channels — the output will match these.
4. Compile the bands to an ffmpeg filterchain:
   - `highpass` → `highpass=f=<freq>`
   - `lowpass` → `lowpass=f=<freq>`
   - `peak` → `equalizer=f=<freq>:t=q:w=<q>:g=<gain>`
   - `lowshelf` → `bass=f=<freq>:g=<gain>:w=<q>`
   - `highshelf` → `treble=f=<freq>:g=<gain>:w=<q>`
   Join with commas.
5. Run:
   ```
   ffmpeg -hide_banner -loglevel warning -i "$INPUT" \
     -af "$FILTERCHAIN" \
     -c:a <match-input-codec> -b:a <match-input-bitrate> \
     -ar <match-sr> -ac <match-channels> \
     "$OUTPUT"
   ```
   If the input is a lossless format, keep lossless (no `-b:a`).
6. Report preset used, filterchain applied, input vs output LUFS (via `ebur128`), and output path.

## Notes

- Applying EQ before `/prep-for-transcription` can help if the source has heavy rumble or mud.
- Chain with `/apply-noise-removal` first (denoise → EQ → convert) for best results on rough recordings.
