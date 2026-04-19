---
description: Embed a cover art image into an audio file (ID3 APIC for MP3, metadata block for FLAC/Opus/M4A)
argument-hint: <audio-file> <image-file> [--output <path>]
---

Attach cover art `$2` to audio file `$1`. By default writes `<name>_art.<ext>` (or in-place with `--in-place`).

## Steps

1. `ffprobe` the audio to determine container (mp3, m4a, flac, opus, ogg).
2. Validate image is JPEG or PNG, ≤ 3000×3000 px, sensible size (<5 MB). If too large, offer to resize via ImageMagick before embedding.
3. Embed per container:
   - **MP3 / M4A**:
     ```
     ffmpeg -i "$AUDIO" -i "$IMAGE" -map 0:a -map 1 \
       -c copy -c:v:1 mjpeg \
       -disposition:v:1 attached_pic \
       -metadata:s:v title="Album cover" \
       -metadata:s:v comment="Cover (front)" \
       "$OUTPUT"
     ```
   - **FLAC**:
     ```
     ffmpeg -i "$AUDIO" -i "$IMAGE" -map 0 -map 1 \
       -c copy -disposition:v attached_pic "$OUTPUT"
     ```
   - **Opus / Ogg**: no native cover; use `opustags` or `metaflac`-style base64-in-METADATA_BLOCK_PICTURE. If `opustags` unavailable, tell user how to install (`sudo apt install opus-tools`).
4. Verify embedded art by re-probing output.
5. Report output path and embedded image dimensions/size.
