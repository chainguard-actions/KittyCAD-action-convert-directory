<!-- markdownlint-disable -->

# Hardening Report: KittyCAD--action-convert-directory/v0.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **KittyCAD--action-convert-directory/v0.0.2** was hardened automatically. 6 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'convert files' run: block directly interpolates ${{ inputs.* }} expressions inside shell commands, violating sub-rule (a). Specifically:
- Line 26: `mkdir -p "${{ inputs.output-directory }}"` — inputs.output-directory interpolated directly into shell.
- Line 27: `FILES_TO_CONVERT=$(find ${{ inputs.input-directory }} ...)` — inputs.input-directory interpolated directly and unquoted, allowing shell metacharacter injection.
- Line 34: `RESULT=$(kittycad file convert $FILE "${{ inputs.output-directory }}/${BASENAME%.*}.${{ inputs.conversion-type }}")` — both inputs.output-directory and inputs.conversion-type interpolated directly into shell.
An attacker controlling these inputs (e.g. via workflow_dispatch or a calling workflow) can inject arbitrary shell commands. The fix is to pass inputs via env: variables and double-quote all expansions.

Locations:

- `action.yml:26`
- `action.yml:27`
- `action.yml:34`

### unpinned-uses (severity: high)

The action references `KittyCAD/action-install-cli@v0.1.1` using a mutable version tag instead of a full 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling supply-chain attacks. Pin to a specific commit SHA, e.g. `KittyCAD/action-install-cli@<40-char-sha> # v0.1.1`.

Locations:

- `action.yml:23`

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

Fixed two categories of findings in action.yml:
1. unpinned-uses: Pinned `KittyCAD/action-install-cli@v0.1.1` to full commit SHA `e26335459a56f191468e88c61d66f3f0b7d5dd79` with the tag preserved as a comment.
2. script-injection / static-inline-injection (6 findings): Moved all `${{ inputs.* }}` expressions out of the `run:` shell block into the step's `env:` map. Added INPUT_DIRECTORY, OUTPUT_DIRECTORY, and CONVERSION_TYPE env vars. The shell script now references these as plain environment variables ($INPUT_DIRECTORY, $OUTPUT_DIRECTORY, $CONVERSION_TYPE), properly double-quoted where needed, eliminating all shell injection vectors. The existing KITTYCAD_TOKEN env var was preserved.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted shell variable expansions in action.yml:
1. Replaced the broken `FILES_TO_CONVERT=$(find ...)` + `for FILE in ${FILES_TO_CONVERT[@]}` pattern (FILES_TO_CONVERT was a plain string, not an array) with `mapfile -d '' FILES_TO_CONVERT < <(find ... -print0)` + `for FILE in "${FILES_TO_CONVERT[@]}"` to correctly handle filenames with spaces and shell metacharacters.
2. Added double-quotes around `$FILE` in `basename "$FILE"`.
3. Added double-quotes around `$FILE` in `kittycad file convert "$FILE" ...`.
The env: block already correctly maps all inputs to environment variables, so no ${{ }} expressions appear directly in the run: shell script.

