---
description: Apply a voice-tuned dynamic range compressor preset to smooth out loud/quiet swings
argument-hint: <input-file> [--preset gentle|podcast|broadcast|aggressive] [--makeup auto|<dB>]
---

Compress `$1` with a voice preset and write `<name>_comp.<ext>`.

## Presets

| preset | threshold | ratio | attack (ms) | release (ms) | knee | use case |
|---|---|---|---|---|---|---|
| gentle | -18 dB | 2:1 | 20 | 250 | 2.8 | subtle leveling |
| podcast (default) | -20 dB | 3:1 | 10 | 200 | 2.8 | interview/solo podcast |
| broadcast | -18 dB | 4:1 | 5 | 150 | 2.8 | FM/voiceover tightness |
| aggressive | -24 dB | 6:1 | 3 | 100 | 2.0 | very uneven field recording |

## Steps

1. Choose preset. Build filter:
   ```
   acompressor=threshold=<th>dB:ratio=<r>:attack=<a>:release=<rel>:knee=<k>:makeup=<m>
   ```
   If `--makeup auto`, measure pre/post RMS with `volumedetect` across a 30 s sample and compute the gain needed to match pre-compression integrated loudness.
2. Run:
   ```
   ffmpeg -i "$INPUT" -af "$FILTER" \
     -c:a <match-input-codec> -b:a <match-input-bitrate> \
     "$OUTPUT"
   ```
3. Output naming: same dir, stem + `_comp` + same extension.
4. Report before/after peak, RMS, and integrated LUFS.

Typical chain for voice: `/apply-noise-removal` → `/deess` → `/apply-eq` → `/compress-voice` → `/normalize-audio` or `/master-audio`.
