---
description: Install Silero VAD on this Ubuntu/Linux system for voice activity detection
---

Install Silero VAD so later commands can cut silence from audio files.

## Steps

1. Confirm `ffmpeg` is installed (`which ffmpeg`). If missing, `sudo apt install -y ffmpeg`.
2. Ensure `pipx` is available (`which pipx` or `sudo apt install -y pipx && pipx ensurepath`).
3. Install Silero VAD in an isolated env:
   ```
   pipx install --include-deps silero-vad
   ```
   If `silero-vad` is not a CLI package, fall back to installing the helper:
   ```
   pipx install --include-deps --spec "git+https://github.com/snakers4/silero-vad" silero-vad
   ```
   Or create a small CLI wrapper at `~/.local/bin/silero-vad` that uses the `silero-vad` Python package to emit speech timestamps.
4. Verify: run the CLI with `--help` or import `from silero_vad import load_silero_vad` in a Python one-liner.
5. Update the plugin state file at `${CLAUDE_PLUGIN_ROOT}/data/tools.json` — set `silero_vad.installed = true` and record the resolved `path` and `version`.
6. Report success and the resolved binary path.

Do not use `sudo pip`. Never touch system Python. Prefer `pipx` or a venv under `~/.local/share/audio-editing-plugin/venv`.
