---
description: Master audio to a specific platform's loudness target using EBU R128 two-pass loudnorm
argument-hint: <input-file> [--target spotify|apple-podcast|youtube|amazon-music|tidal|broadcast-ebu|broadcast-us]
---

Two-pass master `$1` to platform loudness spec and write `<name>_mastered-<target>.<ext>`.

## Targets

| target | integrated LUFS | true peak | loudness range |
|---|---|---|---|
| spotify | -14 | -1 dBTP | 11 LU |
| apple-podcast (default) | -16 | -1 dBTP | 10 LU |
| youtube | -14 | -1 dBTP | 11 LU |
| amazon-music | -14 | -2 dBTP | 11 LU |
| tidal | -14 | -1 dBTP | 11 LU |
| broadcast-ebu | -23 | -1 dBTP | 7 LU |
| broadcast-us (ATSC A/85) | -24 | -2 dBTP | 7 LU |

## Steps

1. **Pass 1** — measure:
   ```
   ffmpeg -i "$INPUT" -af "loudnorm=I=<I>:TP=<TP>:LRA=<LRA>:print_format=json" \
     -f null - 2>&1 | tee /tmp/loudnorm-pass1.log
   ```
   Parse the `input_i`, `input_tp`, `input_lra`, `input_thresh`, `target_offset` JSON block.
2. **Pass 2** — apply with measured values for linear normalization (no dynamics crush):
   ```
   ffmpeg -i "$INPUT" -af \
     "loudnorm=I=<I>:TP=<TP>:LRA=<LRA>:measured_I=<mi>:measured_TP=<mtp>:measured_LRA=<mlra>:measured_thresh=<mth>:offset=<off>:linear=true:print_format=summary" \
     -c:a <match-input-codec> -b:a <match-input-bitrate> \
     -ar <match-sr> -ac <match-channels> \
     "$OUTPUT"
   ```
3. Output naming: same dir, stem + `_mastered-<target>` + same extension.
4. Verify with a final `ebur128` pass and report achieved LUFS / TP / LRA.

## Notes

- Use after `/compress-voice` and `/apply-eq`, not before.
- For music, keep LRA ≥ 9; for voice/podcast, LRA 6–10 is typical.
- If the file is already louder than target, loudnorm will attenuate without adding a limiter — safe.
