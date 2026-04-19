---
description: Change bitrate (and optionally sample rate / channels / codec) of an audio file via ffmpeg
argument-hint: <input-file> [--bitrate 64k] [--sample-rate 48000] [--channels 2] [--codec opus]
---

Re-encode `$1` with the requested parameters and write `<name>_<bitrate>.<ext>` in the same directory (extension matches the chosen codec).

## Steps

1. `ffprobe` the input — report current codec, bitrate, sample rate, channels, duration.
2. Resolve flags. If the user passed only `--bitrate`, keep original sample rate, channels, and codec. If `--codec` is set, pick the matching file extension (`opus`→`.opus`, `aac`→`.m4a`, `mp3`→`.mp3`, `flac`→`.flac`, `pcm_s16le`→`.wav`).
3. Build ffmpeg command. Examples:
   - Opus: `ffmpeg -i "$INPUT" -c:a libopus -b:a "$BITRATE" -ar "$SR" -ac "$CH" "$OUTPUT"`
   - AAC: `ffmpeg -i "$INPUT" -c:a aac -b:a "$BITRATE" "$OUTPUT"`
   - MP3: `ffmpeg -i "$INPUT" -c:a libmp3lame -b:a "$BITRATE" "$OUTPUT"`
4. Name output with the bitrate in the stem, e.g. `interview_64k.opus`. If the file already exists, append a counter.
5. Report old vs new size, bitrate, codec, and output path.

## Notes

- Opus below 24 kbps degrades fast; below 16 kbps is speech-only territory.
- For music, prefer `--codec opus --bitrate 96k` or higher.
- For voice archival, 32–48 kbps Opus mono is typically transparent.
