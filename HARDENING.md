<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.11.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's single `run:` step (action.yml, step starting at line 36) directly interpolates multiple GitHub Actions expressions inside shell commands, violating rule (a). This allows script injection via attacker-controlled inputs and context values:

- Line 37: `pushd '${{ github.action_path }}' && composer i -n && popd` — github.action_path interpolated directly
- Line 39: `if [ "${{ inputs.report_enabled }}" = "true" ]; then` — inputs.report_enabled interpolated directly
- Line 42: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — inputs.owner interpolated directly
- Line 43: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — inputs.repo interpolated directly
- Line 44: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — inputs.issue_number interpolated directly
- Line 45: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — inputs.token interpolated directly
- Line 49: `BRANCH_BASE="origin/${{ inputs.base }}"` — inputs.base interpolated directly; an attacker-controlled branch name could inject shell metacharacters
- Line 50: `BRANCH_HEAD="${{ github.sha }}"` — github.sha interpolated directly
- Line 52: `${{ github.action_path }}/bin/loxcan ...` — github.action_path used as command prefix

All ${{ ... }} expressions are substituted by the Actions runner before the shell ever sees the script, so any expression containing shell metacharacters (;, |, &, $(...), etc.) will be executed. The fix is to move all values into env: variables and reference them as quoted shell variables inside the run: block.

Locations:

- `action.yml:37`
- `action.yml:39`
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

Moved all ${{ }} expressions from the run: block into a new env: block on the composite action step. The following mappings were created: ACTION_PATH=${{ github.action_path }}, INPUT_REPORT_ENABLED=${{ inputs.report_enabled }}, INPUT_OWNER=${{ inputs.owner }}, INPUT_REPO=${{ inputs.repo }}, INPUT_ISSUE_NUMBER=${{ inputs.issue_number }}, INPUT_TOKEN=${{ inputs.token }}, INPUT_BASE=${{ inputs.base }}, GITHUB_SHA_VALUE=${{ github.sha }}. The shell script was updated to reference these as quoted environment variables (e.g., "$ACTION_PATH", "$INPUT_OWNER", etc.), and the loxcan binary invocation was changed from using ${{ github.action_path }} as a command prefix to using the safe "$ACTION_PATH/bin/loxcan" form. This eliminates all shell injection vectors identified in the findings.

