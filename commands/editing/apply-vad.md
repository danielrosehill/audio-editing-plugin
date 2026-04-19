---
description: Run Silero VAD on an audio file and write a silence-cut version with _vad suffix
argument-hint: <input-file> [--aggressiveness 0.5] [--min-speech-ms 250] [--pad-ms 150]
---

Apply Silero VAD to `$1` and produce `<name>_vad.<ext>` in the same directory, containing only detected speech segments concatenated back together.

## Steps

1. Read `${CLAUDE_PLUGIN_ROOT}/data/tools.json`. If `silero_vad.installed` is false, tell the user to run `/install-vad` first and stop.
2. Resolve the input path from `$1`. Compute output: same dir, same stem + `_vad` + same extension. If the output exists, refuse unless the user passes `--force`.
3. Run VAD to get speech timestamps. Use a small Python script (via the Silero env) that:
   - Loads audio with `torchaudio` at 16 kHz mono (resample in memory only, keep original sample rate for output).
   - Calls `get_speech_timestamps(audio, model, threshold=<aggressiveness>, min_speech_duration_ms=<min-speech-ms>)`.
   - Pads each segment by `<pad-ms>` on both sides, merges overlaps.
   - Emits segment ranges as a newline-separated `start,end` list (seconds) on stdout.
4. Build an ffmpeg `concat` plan from the segments and write to the output file preserving the input codec and bitrate:
   ```
   ffmpeg -hide_banner -loglevel warning -i "$INPUT" \
     -filter_complex "<aselect+asetpts chain built from segments>" \
     -c:a copy-or-reencode-matching-input "$OUTPUT"
   ```
   If `-c:a copy` is not compatible with the filter chain, re-encode to match the input codec and bitrate (`ffprobe` the input first).
5. Report input duration, output duration, percent silence removed, and output path.

## Defaults

- `aggressiveness`: 0.5 (Silero threshold)
- `min-speech-ms`: 250
- `pad-ms`: 150

Do not alter sample rate, channel count, or codec unless the user explicitly asks — use `/prep-for-transcription` for that.
