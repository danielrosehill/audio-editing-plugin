---
description: Save an EQ curve as a named user preset in plugin data and (optionally) to ~/.config/audio-editing/eq/
argument-hint: <preset-name> [--description "..."] [--from-last-recommendation]
---

Persist an EQ preset so it can be reused via `/apply-eq`.

## Steps

1. Resolve the bands JSON:
   - If `--from-last-recommendation`, pull the last `/recommend-eq` output from the conversation.
   - Otherwise, ask the user to paste bands JSON (same schema as `eq-presets.json`) or answer questions to build one.
2. Validate each band:
   - `type` ∈ `highpass | lowpass | peak | lowshelf | highshelf`
   - `freq` in Hz (20–20000)
   - `gain` in dB (for peak/shelf), −24 to +24
   - `q` for peak/shelf (0.1–10)
3. Read `${CLAUDE_PLUGIN_ROOT}/data/eq-presets.json`. Add under `user_presets.<name>`:
   ```json
   {
     "description": "...",
     "created_at": "<ISO timestamp>",
     "bands": [ ... ]
   }
   ```
   Refuse to overwrite an existing user preset unless `--force`.
4. Also mirror the preset to `~/.config/audio-editing/eq/<name>.json` so it survives plugin cache rebuilds.
5. Report the saved path, the name, and the ffmpeg filter string it compiles to.

Built-in presets (podcast, transcription, broadcast, intimate) must not be overwritten.
