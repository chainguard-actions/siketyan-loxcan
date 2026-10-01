<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.9.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates multiple `${{ }}` expressions into shell commands (sub-rule a), allowing script injection. Attacker-controllable inputs are embedded verbatim into the shell script before the shell ever sees them:

- Line 35: `pushd '${{ github.action_path }}'` — github context interpolated directly
- Line 37: `if [ "${{ inputs.report_enabled }}" = "true" ]` — inputs interpolated directly
- Line 41: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — inputs interpolated directly
- Line 42: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — inputs interpolated directly
- Line 43: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — inputs interpolated directly
- Line 44: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — inputs interpolated directly
- Line 48: `BRANCH_BASE="origin/${{ inputs.base }}"` — attacker-controlled branch name interpolated directly into shell
- Line 49: `BRANCH_HEAD="${{ github.sha }}"` — github context interpolated directly
- Line 51: `${{ github.action_path }}/bin/loxcan` — github context used as command path

Additionally (sub-rule b), line 51 uses `${LOXCAN_ARGS}` unquoted, which allows shell metacharacter splitting. All `${{ inputs.* }}` and `${{ github.* }}` values should be passed via `env:` variables and referenced as quoted shell variables (e.g., `"$INPUT_OWNER"`) instead of being interpolated directly.

Locations:

- `action.yml:34`

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

Moved all ${{ }} expressions from the run: block into the step's env: block. Each expression is now referenced as a quoted shell variable ($VAR_NAME) in the script. The unquoted ${LOXCAN_ARGS} variable was replaced with a bash array (loxcan_args) expanded as "${loxcan_args[@]}" to keep argument boundaries intact. Affected expressions: github.action_path, inputs.report_enabled, inputs.owner, inputs.repo, inputs.issue_number, inputs.token, inputs.base, github.sha.

