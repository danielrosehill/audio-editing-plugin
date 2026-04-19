---
description: Analyze an audio sample's spectrum and propose an EQ curve tailored to the voice and target use case
argument-hint: <sample-file> [--target podcast|transcription|broadcast|intimate] [--voice-type male-low|male-mid|female-mid|female-high]
---

Inspect the spectral profile of `$1` and recommend an EQ preset. Do not modify the file.

## Steps

1. Probe with `ffprobe` — duration, SR, channels, codec.
2. Extract spectral statistics with ffmpeg:
   ```
   ffmpeg -hide_banner -i "$INPUT" -af \
     "aspectralstats=measure=centroid+spread+flatness+rolloff+kurtosis,\
      astats=metadata=1:reset=0,ebur128=peak=true" \
     -f null - 2>&1 | tee /tmp/eq-analysis.log
   ```
   Also generate a spectrogram PNG for visual reference:
   ```
   ffmpeg -y -i "$INPUT" -lavfi \
     "showspectrumpic=s=1600x800:mode=combined:color=intensity:scale=log" \
     /tmp/eq-spectrogram.png
   ```
3. Parse mean centroid, spread, low/high rolloff, integrated LUFS, true peak. Summarize:
   - Fundamental / formant region (expected by voice type)
   - Low-end rumble (below 80 Hz) — is there energy to remove?
   - Muddy range (200–400 Hz) — excess buildup?
   - Presence (2–5 kHz) — lacking or harsh?
   - Sibilance (5–9 kHz) — problematic?
   - Air (10 kHz+) — present or rolled off?
4. Cross-reference the `--target` (default: podcast). Start from the matching builtin in `${CLAUDE_PLUGIN_ROOT}/data/eq-presets.json` and adjust bands based on what the spectral stats revealed.
5. Output a proposed EQ as JSON in the same schema as `eq-presets.json` bands, plus a 2–3 sentence rationale per band. Also show the ffmpeg filter string the user would apply:
   ```
   highpass=f=80,equalizer=f=250:t=q:w=1:g=-2,equalizer=f=2500:t=q:w=1.2:g=2, ...
   ```
6. Offer next steps:
   - `/save-eq <name>` to persist this recommendation
   - `/apply-eq <name> <file>` to apply it

Never auto-save. Never modify the input. The spectrogram path and analysis log stay in `/tmp` for the user to review.
