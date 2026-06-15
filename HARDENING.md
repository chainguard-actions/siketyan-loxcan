<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **siketyan--loxcan/v0.10.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` step in action.yml directly interpolates multiple `${{ }}` expressions into shell commands before the shell parses them, enabling script injection (sub-rule a). Offending lines include:
- Line 36: `pushd '${{ github.action_path }}' && composer i -n && popd`
- Line 38: `if [ "${{ inputs.report_enabled }}" = "true" ]; then`
- Line 42: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"`
- Line 43: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"`
- Line 44: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"`
- Line 45: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"`
- Line 49: `BRANCH_BASE="origin/${{ inputs.base }}"`
- Line 50: `BRANCH_HEAD="${{ github.sha }}"`
- Line 52: `${{ github.action_path }}/bin/loxcan ${LOXCAN_ARGS} "${BRANCH_BASE}" "${BRANCH_HEAD}"`

Attacker-controlled inputs (`inputs.owner`, `inputs.repo`, `inputs.issue_number`, `inputs.token`, `inputs.base`, `inputs.report_enabled`) and GitHub context values (`github.action_path`, `github.sha`) are all substituted directly into the shell script by the Actions runner before the shell executes it. A malicious value such as `inputs.base` set to `main"; malicious_command; echo "` would execute arbitrary commands. All `${{ }}` expressions must be moved to `env:` variables and referenced as quoted shell variables instead.

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

Moved all ${{ }} expressions (github.action_path, inputs.report_enabled, inputs.owner, inputs.repo, inputs.issue_number, inputs.token, inputs.base, github.sha) from the run: shell script into an env: block on the step. Each expression is now assigned to a named environment variable (ACTION_PATH, REPORT_ENABLED, INPUT_OWNER, INPUT_REPO, INPUT_ISSUE_NUMBER, INPUT_TOKEN, INPUT_BASE, GIT_SHA) and referenced as a quoted shell variable in the run: block, preventing script injection attacks.

