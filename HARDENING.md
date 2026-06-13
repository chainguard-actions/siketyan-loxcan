<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **siketyan--loxcan/v0.11.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple GitHub Actions expressions (${{ ... }}) inside shell command strings, violating rule (a). This allows an attacker to inject arbitrary shell commands via controlled inputs or github context values. Affected lines:
- Line 36: `pushd '${{ github.action_path }}' && composer i -n && popd` — github.action_path interpolated directly
- Line 38: `if [ "${{ inputs.report_enabled }}" = "true" ]` — inputs.report_enabled interpolated directly
- Line 41: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — inputs.owner interpolated directly
- Line 42: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — inputs.repo interpolated directly
- Line 43: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — inputs.issue_number interpolated directly
- Line 44: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — inputs.token interpolated directly
- Line 48: `BRANCH_BASE="origin/${{ inputs.base }}"` — inputs.base interpolated directly
- Line 49: `BRANCH_HEAD="${{ github.sha }}"` — github.sha interpolated directly
- Line 51: `${{ github.action_path }}/bin/loxcan ${LOXCAN_ARGS} ...` — github.action_path interpolated directly AND ${LOXCAN_ARGS} is unquoted (rule b violation)
All ${{ inputs.* }} and ${{ github.* }} values should be passed via env: variables and then referenced as double-quoted shell variables.

Locations:

- `action.yml:36`
- `action.yml:38`
- `action.yml:41`
- `action.yml:42`
- `action.yml:43`
- `action.yml:44`
- `action.yml:48`
- `action.yml:49`
- `action.yml:51`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.report_enabled }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:39`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.owner }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:43`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.issue_number }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.base }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all ${{ github.* }} and ${{ inputs.* }} expressions from the run: block into a step-level env: block. Variables introduced: ACTION_PATH (github.action_path), REPORT_ENABLED (inputs.report_enabled), INPUT_OWNER (inputs.owner), INPUT_REPO (inputs.repo), INPUT_ISSUE_NUMBER (inputs.issue_number), INPUT_TOKEN (inputs.token), INPUT_BASE (inputs.base), GITHUB_SHA_VALUE (github.sha). All shell references are now double-quoted environment variables. The optional LOXCAN_ARGS argument uses ${LOXCAN_ARGS:+"$LOXCAN_ARGS"} so it drops out entirely when unset, preserving correct argument count. The loxcan binary path is now "$ACTION_PATH/bin/loxcan" (double-quoted) instead of a direct ${{ }} interpolation.

