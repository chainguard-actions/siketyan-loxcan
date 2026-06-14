<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **siketyan--loxcan/v0.9.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` block in action.yml directly interpolates multiple GitHub Actions expressions into shell commands (rule a). Before the shell executes the script, YAML template substitution replaces these expressions with their raw values, allowing an attacker-controlled input to inject arbitrary shell commands.

Offending lines:
- Line 35: `pushd '${{ github.action_path }}' && composer i -n && popd` — `github.action_path` interpolated directly
- Line 37: `if [ "${{ inputs.report_enabled }}" = "true" ]; then` — `inputs.report_enabled` interpolated directly
- Line 41: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — `inputs.owner` interpolated directly
- Line 42: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — `inputs.repo` interpolated directly
- Line 43: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — `inputs.issue_number` interpolated directly
- Line 44: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — `inputs.token` interpolated directly
- Line 47: `BRANCH_BASE="origin/${{ inputs.base }}"` — `inputs.base` interpolated directly
- Line 48: `BRANCH_HEAD="${{ github.sha }}"` — `github.sha` interpolated directly
- Line 50: `${{ github.action_path }}/bin/loxcan ...` — `github.action_path` used as command prefix

All `inputs.*` values are caller-controlled and must be passed via `env:` variables and then referenced as `"$VAR"` in the shell, never via `${{ ... }}` inside a `run:` block.

Locations:

- `action.yml:35`
- `action.yml:37`
- `action.yml:41`
- `action.yml:42`
- `action.yml:43`
- `action.yml:44`
- `action.yml:47`
- `action.yml:48`
- `action.yml:50`

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

Fixed all script injection findings in action.yml by moving all ${{ }} expressions from the run: block into an env: block. Specifically: github.action_path → ACTION_PATH, inputs.report_enabled → INPUT_REPORT_ENABLED, inputs.owner → INPUT_OWNER, inputs.repo → INPUT_REPO, inputs.issue_number → INPUT_ISSUE_NUMBER, inputs.token → INPUT_TOKEN, inputs.base → INPUT_BASE, github.sha → GITHUB_SHA_VALUE. The shell script now references these as plain environment variables. The action_path used as a command prefix is now safely quoted as "$ACTION_PATH/bin/loxcan".

