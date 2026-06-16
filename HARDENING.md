<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **siketyan--loxcan/v0.10.1** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's `run:` block in action.yml directly interpolates multiple `${{ }}` expressions into shell commands (rule a). This allows an attacker to inject arbitrary shell commands by controlling the input values or via crafted GitHub event data. Affected lines:
- Line 36: `pushd '${{ github.action_path }}'` — github context interpolated directly into shell
- Line 38: `if [ "${{ inputs.report_enabled }}" = "true" ]` — attacker-controlled input interpolated directly
- Line 42: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — attacker-controlled input
- Line 43: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — attacker-controlled input
- Line 44: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — attacker-controlled input
- Line 45: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — attacker-controlled input
- Line 49: `BRANCH_BASE="origin/${{ inputs.base }}"` — attacker-controlled input
- Line 50: `BRANCH_HEAD="${{ github.sha }}"` — github context interpolated directly
- Line 52: `${{ github.action_path }}/bin/loxcan ...` — github context used as command prefix

All these values should be passed via `env:` variables and then referenced as properly double-quoted shell variables (e.g., `"$INPUT_BASE"`) instead of being interpolated directly as `${{ }}` expressions.

Locations:

- `action.yml:36`
- `action.yml:38`
- `action.yml:42`
- `action.yml:43`
- `action.yml:44`
- `action.yml:45`
- `action.yml:49`
- `action.yml:50`
- `action.yml:52`

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

Fixed all script injection findings in action.yml by moving all ${{ }} expressions (github.action_path, github.sha, inputs.report_enabled, inputs.owner, inputs.repo, inputs.issue_number, inputs.token, inputs.base) from the run: block into a step-level env: map. The shell script now references these values as plain environment variables (e.g., $ACTION_PATH, $INPUT_REPORT_ENABLED, $INPUT_OWNER, etc.) instead of directly interpolating ${{ }} expressions. The binary invocation was also changed from `${{ github.action_path }}/bin/loxcan` to `"$ACTION_PATH/bin/loxcan"` (properly quoted) to prevent both injection and word-splitting issues.

