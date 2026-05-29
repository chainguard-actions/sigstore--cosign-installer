# Hardening Report: sigstore--cosign-installer/v4.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **sigstore--cosign-installer/v4.1.2** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Two steps write the attacker-controlled `inputs.install-dir` value to `$GITHUB_PATH` via an env var (`input_install_dir`) without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). 

**Step 2 (Linux/macOS):** `envsubst <<< "${input_install_dir}" >> "$GITHUB_PATH"` — the value is expanded and appended directly to GITHUB_PATH with no newline stripping.

**Step 3 (Windows):** `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH` — the expanded value is written directly to GITHUB_PATH with no sanitization.

An attacker supplying a newline-containing value for `install-dir` could inject arbitrary entries into `$GITHUB_PATH`, potentially hijacking subsequent command lookups.

Locations:

- `action.yml:243`
- `action.yml:250`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two steps that wrote the attacker-controlled `inputs.install-dir` value to `$GITHUB_PATH` without sanitization:

1. **Linux/macOS step**: Changed `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` to a multi-line script that pipes through `tr -d '\n\r'` to strip newlines before writing to GITHUB_PATH via `printf '%s\n'`.

2. **Windows step**: Added `$safe_install_dir = $install_dir -replace '[\r\n]', ''` to strip newline and carriage-return characters from the expanded value before writing to GITHUB_PATH.

Both fixes prevent an attacker from injecting arbitrary entries into `$GITHUB_PATH` by supplying a newline-containing value for the `install-dir` input.

