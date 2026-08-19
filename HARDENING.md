<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.10.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The single `run:` step in action.yml directly interpolates multiple `${{ }}` expressions inside the shell script (rule a). Before the shell ever parses the command, GitHub Actions substitutes these values as raw text, allowing an attacker who controls the calling workflow's inputs or event payload to inject arbitrary shell commands.

Offending lines:
- Line 35: `pushd '${{ github.action_path }}' && composer i -n && popd`
- Line 37: `if [ "${{ inputs.report_enabled }}" = "true" ]; then`
- Line 40: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"`
- Line 41: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"`
- Line 42: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"`
- Line 43: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"`
- Line 46: `BRANCH_BASE="origin/${{ inputs.base }}"`
- Line 47: `BRANCH_HEAD="${{ github.sha }}"`
- Line 49: `${{ github.action_path }}/bin/loxcan ${LOXCAN_ARGS} "${BRANCH_BASE}" "${BRANCH_HEAD}"`

Fix: Move all `${{ inputs.* }}` and `${{ github.* }}` values into `env:` variables on the step, then reference them as `"$ENV_VAR"` (double-quoted) inside the shell script. For example:
```yaml
env:
  ACTION_PATH: ${{ github.action_path }}
  REPORT_ENABLED: ${{ inputs.report_enabled }}
  OWNER: ${{ inputs.owner }}
  REPO: ${{ inputs.repo }}
  ISSUE_NUMBER: ${{ inputs.issue_number }}
  TOKEN: ${{ inputs.token }}
  BASE: ${{ inputs.base }}
  GH_SHA: ${{ github.sha }}
run: |
  pushd "$ACTION_PATH" && composer i -n && popd
  if [ "$REPORT_ENABLED" = "true" ]; then
    ...
  fi
  BRANCH_BASE="origin/$BASE"
  BRANCH_HEAD="$GH_SHA"
  "$ACTION_PATH"/bin/loxcan ${LOXCAN_ARGS} "$BRANCH_BASE" "$BRANCH_HEAD"
```

Locations:

- `action.yml:35`
- `action.yml:37`
- `action.yml:40`
- `action.yml:41`
- `action.yml:42`
- `action.yml:43`
- `action.yml:46`
- `action.yml:47`
- `action.yml:49`

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

Moved all ${{ }} expressions (github.action_path, inputs.report_enabled, inputs.owner, inputs.repo, inputs.issue_number, inputs.token, inputs.base, github.sha) out of the run: shell script and into an env: block on the step. The shell script now references only plain environment variables (ACTION_PATH, REPORT_ENABLED, OWNER, REPO, ISSUE_NUMBER, TOKEN, BASE, GH_SHA) with proper double-quoting, eliminating all shell injection vectors.

