---
description: Update audio metadata tags (title, artist, album, date, track, comment, etc.)
argument-hint: <audio-file> [--title "..."] [--artist "..."] [--album "..."] [--date 2026] [--track 1/12] [--genre "..."] [--comment "..."]
---

Update metadata on `$1`. Writes to `<name>_tagged.<ext>` unless `--in-place`.

## Steps

1. `ffprobe -show_format` the input to display current tags.
2. For each flag the user passed, add `-metadata <key>="<value>"` to the ffmpeg command:
   ```
   ffmpeg -i "$INPUT" -c copy \
     -metadata title="..." \
     -metadata artist="..." \
     -metadata album="..." \
     -metadata date="..." \
     -metadata track="..." \
     -metadata genre="..." \
     -metadata comment="..." \
     "$OUTPUT"
   ```
3. Container-specific keys:
   - MP3 (ID3v2): `TIT2`, `TPE1`, `TALB`, `TDRC`, `TRCK` — ffmpeg maps standard keys automatically.
   - M4A: uses `©nam`, `©ART`, etc. — handled by ffmpeg via standard keys.
   - Opus/Vorbis: uppercase Vorbis comments (`TITLE`, `ARTIST`, `ALBUM`).
4. Preserve existing tags the user didn't specify (ffmpeg does this by default with `-c copy`).
5. Verify by re-probing the output and diff against input tags. Report changes.

Supports reading values from a JSON/YAML manifest with `--from <manifest.json>` if the user passes one, for batch tagging.
