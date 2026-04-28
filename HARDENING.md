# Hardening Report: sigstore--cosign-installer/v4.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

Action **sigstore--cosign-installer/v4.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The Linux/macOS step writes the attacker-controlled `inputs.install-dir` value (via env var `input_install_dir`) directly to `$GITHUB_PATH` using `envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary entries into GITHUB_PATH, enabling path-hijacking attacks.

Locations:

- `action.yml:234`

### github-env-injection (severity: high)

The Windows step writes the attacker-controlled `inputs.install-dir` value (via env var `input_install_dir`) directly to `$env:GITHUB_PATH` using `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` without sanitization. A malicious value containing newlines could inject arbitrary entries into GITHUB_PATH, enabling path-hijacking attacks.

Locations:

- `action.yml:241`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed both github-env-injection findings in action.yml:

1. Linux/macOS step (was line 234): Replaced single-line `run: envsubst <<<"${input_install_dir}" >> "$GITHUB_PATH"` with a multi-line block that captures envsubst output, strips newlines/carriage returns via `printf '%s' "$expanded" | tr -d '\n\r'`, then writes the sanitized value to $GITHUB_PATH.

2. Windows step (was line 241): Replaced `echo "${install_dir}" | Out-File -FilePath $env:GITHUB_PATH -Encoding utf8 -Append` with PowerShell that strips newlines using `$install_dir -replace '[\r\n]', ''` before writing to $GITHUB_PATH via `Add-Content`.

Both fixes prevent path-hijacking attacks where a malicious `inputs.install-dir` value containing embedded newlines could inject arbitrary entries into GITHUB_PATH.

