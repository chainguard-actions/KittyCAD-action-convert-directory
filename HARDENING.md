<!-- markdownlint-disable -->

# Hardening Report: KittyCAD--action-convert-directory/v0.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **KittyCAD--action-convert-directory/v0.0.2** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block directly interpolates multiple `${{ inputs.* }}` expressions inside shell commands (sub-rule a). Specifically:
- `mkdir -p "${{ inputs.output-directory }}"` — input interpolated directly into a shell command
- `find ${{ inputs.input-directory }} -type f ...` — input interpolated unquoted into a shell command
- `kittycad file convert $FILE "${{ inputs.output-directory }}/${BASENAME%.*}.${{ inputs.conversion-type }}"` — two inputs interpolated directly

All of these values are attacker-controlled and flow through YAML template substitution before the shell parses them, enabling arbitrary command injection. Additionally (sub-rule b), `$FILE` and `${FILES_TO_CONVERT[@]}` are expanded unquoted in the `for` loop and `basename` call, allowing shell metacharacter injection from filenames.

Locations:

- `action.yml:28`

### unpinned-uses (severity: high)

The composite action step `uses: KittyCAD/action-install-cli@v0.1.1` references a mutable version tag (`v0.1.1`) instead of a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved or the upstream repository is compromised.

Locations:

- `action.yml:27`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output-directory }}" appears directly in run: block of step "convert files"; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.input-directory }}" appears directly in run: block of step "convert files"; move to env: map

Locations:

- `action.yml:29`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.output-directory }}" appears directly in run: block of step "convert files"; move to env: map

Locations:

- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.conversion-type }}" appears directly in run: block of step "convert files"; move to env: map

Locations:

- `action.yml:36`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

Fixed all findings in action.yml:
1. Pinned KittyCAD/action-install-cli@v0.1.1 to full SHA e26335459a56f191468e88c61d66f3f0b7d5dd79
2. Moved all ${{ inputs.input-directory }}, ${{ inputs.output-directory }}, and ${{ inputs.conversion-type }} expressions out of the run: block into the env: block as INPUT_INPUT_DIRECTORY, INPUT_OUTPUT_DIRECTORY, and INPUT_CONVERSION_TYPE respectively
3. Fixed unquoted variable expansions: $FILE → "$FILE", basename $FILE → basename "$FILE"
4. Replaced the unsafe FILES_TO_CONVERT=$(find ...) array pattern (which suffered from word-splitting on filenames with spaces) with a null-delimited find -print0 | while IFS= read -r -d '' loop for safe filename handling

