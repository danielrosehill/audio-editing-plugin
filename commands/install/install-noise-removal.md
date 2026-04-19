---
description: Install DeepFilterNet (deep-filter) for deep learning based noise/background removal
---

Install DeepFilterNet so later commands can denoise speech recordings.

## Steps

1. Check for an existing `deep-filter` binary (`which deep-filter`). If present, just record its path.
2. If missing, install the prebuilt binary:
   ```
   mkdir -p ~/.local/bin
   cd /tmp
   curl -L -o deep-filter.tar.gz \
     https://github.com/Rikorose/DeepFilterNet/releases/latest/download/deep-filter-x86_64-unknown-linux-musl.tar.gz
   tar xf deep-filter.tar.gz
   install -m 755 deep-filter ~/.local/bin/deep-filter
   ```
   If the release asset name has changed, list releases via `gh release list -R Rikorose/DeepFilterNet` and pick the current Linux x86_64 asset.
3. Fallback: `pipx install deepfilternet` (Python wrapper, slower startup).
4. Verify: `deep-filter --version`.
5. Update `${CLAUDE_PLUGIN_ROOT}/data/tools.json` — set `deep_filter.installed = true` with `path` and `version`.
6. Report success and path.

Never `sudo pip install`. Prefer the static binary; only fall back to pipx if the binary does not run on this system.
