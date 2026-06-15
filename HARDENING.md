<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **siketyan--loxcan/v0.9.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's single `run:` step in action.yml directly interpolates multiple GitHub Actions expressions (`${{ ... }}`) into shell command strings, violating sub-rule (a). Before the shell ever sees the script, YAML template substitution replaces these expressions with their raw values — an attacker-controlled value (e.g. a branch name in `inputs.base`, or a crafted `inputs.owner`) can inject arbitrary shell commands.

Offending interpolations:
- Line 34: `pushd '${{ github.action_path }}'` — action_path used as shell argument
- Line 36: `if [ "${{ inputs.report_enabled }}" = "true" ]` — inputs.report_enabled interpolated in condition
- Line 38: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — inputs.owner interpolated
- Line 39: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — inputs.repo interpolated
- Line 40: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — inputs.issue_number interpolated
- Line 41: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — inputs.token interpolated
- Line 45: `BRANCH_BASE="origin/${{ inputs.base }}"` — inputs.base interpolated (high risk: branch names are attacker-controlled)
- Line 46: `BRANCH_HEAD="${{ github.sha }}"` — github.sha interpolated
- Line 48: `${{ github.action_path }}/bin/loxcan` — action_path used as command prefix

All inputs.* values should be passed via `env:` block variables and referenced as `"$VAR"` in the shell script, never as `${{ ... }}` directly inside `run:`.

Locations:

- `action.yml:34`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.report_enabled }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:39`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.owner }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:41`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.repo }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:42`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.issue_number }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:43`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:44`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.base }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all ${{ ... }} expressions from the run: block into a new env: block on the composite step. Specifically: github.action_path → ACTION_PATH, inputs.report_enabled → REPORT_ENABLED, inputs.owner → INPUT_OWNER, inputs.repo → INPUT_REPO, inputs.issue_number → INPUT_ISSUE_NUMBER, inputs.token → INPUT_TOKEN, inputs.base → INPUT_BASE, github.sha → GITHUB_SHA_VALUE. All shell references updated to use the corresponding $VAR_NAME environment variables. The loxcan binary is now invoked as "$ACTION_PATH/bin/loxcan" (quoted) instead of using the raw ${{ github.action_path }} expression directly as a command prefix.

