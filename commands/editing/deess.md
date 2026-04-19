---
description: Tame sibilance ("s", "sh", "ts" hiss) using ffmpeg's deesser filter
argument-hint: <input-file> [--intensity light|medium|strong] [--freq 6500]
---

Apply de-essing to `$1` and write `<name>_deess.<ext>`.

## Steps

1. Pick threshold / ratio from `--intensity`:
   - `light`: `i=0.3:m=0.5:f=0.5`
   - `medium`: `i=0.5:m=0.5:f=0.5` (default)
   - `strong`: `i=0.8:m=0.5:f=0.5`
2. Run:
   ```
   ffmpeg -i "$INPUT" \
     -af "deesser=i=<i>:m=<m>:f=<f>:s=o" \
     -c:a <match-input-codec> -b:a <match-input-bitrate> \
     "$OUTPUT"
   ```
3. If `deesser` is unavailable on the local ffmpeg build, fall back to a band-specific compressor around `--freq`:
   ```
   -af "highpass=f=5000,acompressor=threshold=-20dB:ratio=6:attack=1:release=50,lowpass=f=9000"
   ```
   Sidechain-style deessing is preferable if the user asks for precision.
4. Output naming: same dir, stem + `_deess` + same extension.
5. Report before/after spectral centroid in the 5–9 kHz band.

Chain `/deess` after `/apply-noise-removal` — denoising can increase perceived sibilance.
