<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.9.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's `run:` block in action.yml directly interpolates multiple `${{ }}` expressions into shell commands (sub-rule a), allowing script injection. Attacker-controllable inputs are interpolated without sanitization:
- Line 35: `pushd '${{ github.action_path }}' && composer i -n && popd` — github.action_path interpolated directly
- Line 37: `if [ "${{ inputs.report_enabled }}" = "true" ]` — inputs.report_enabled interpolated in shell condition
- Line 39: `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"` — inputs.owner interpolated
- Line 40: `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"` — inputs.repo interpolated
- Line 41: `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"` — inputs.issue_number interpolated
- Line 42: `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"` — inputs.token interpolated
- Line 45: `BRANCH_BASE="origin/${{ inputs.base }}"` — inputs.base interpolated
- Line 46: `BRANCH_HEAD="${{ github.sha }}"` — github.sha interpolated
- Line 48: `${{ github.action_path }}/bin/loxcan` — github.action_path used as command prefix

All of these `${{ }}` expressions are substituted by the Actions runner before the shell parses the script, meaning any newlines, shell metacharacters, or injected commands in the values will be executed. The inputs.* values are fully attacker-controlled when the action is called from a workflow. These should be moved to `env:` variables and referenced as `"$ENV_VAR"` in the shell script.

Locations:

- `action.yml:35`
- `action.yml:37`
- `action.yml:39`
- `action.yml:40`
- `action.yml:41`
- `action.yml:42`
- `action.yml:45`
- `action.yml:46`
- `action.yml:48`

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

Moved all ${{ }} expressions from the run: block into an env: block on the composite action step in action.yml. The following mappings were applied:
- ${{ github.action_path }} → ACTION_PATH env var
- ${{ inputs.report_enabled }} → INPUT_REPORT_ENABLED env var
- ${{ inputs.owner }} → INPUT_OWNER env var
- ${{ inputs.repo }} → INPUT_REPO env var
- ${{ inputs.issue_number }} → INPUT_ISSUE_NUMBER env var
- ${{ inputs.token }} → INPUT_TOKEN env var
- ${{ inputs.base }} → INPUT_BASE env var
- ${{ github.sha }} → GITHUB_SHA_VALUE env var (named to avoid collision with the built-in GITHUB_SHA)

All shell references were updated to use the corresponding $ENV_VAR names. The loxcan binary invocation was also fixed to use "$ACTION_PATH/bin/loxcan" (quoted, as a single path) instead of the raw expression as a command prefix.

### Iteration 2

**Fixes applied:** unpinned-uses

**Notes:**

Pinned all 7 unpinned `uses:` references to immutable full 40-character commit SHAs across 3 workflow files:
- .github/workflows/action.yml: actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744
- .github/workflows/docker.yml: actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744, docker/setup-buildx-action@v2 → @885d1462b80bc1c1c7f0b00334ad271f09369c55, docker/login-action@v2 → @465a07811f14bebb1938fbed4728c6a1ff8901fc, docker/build-push-action@v4 → @0a97817b6ade9f46837855d676c4cca3a2471fc9
- .github/workflows/php.yml: actions/checkout@v3 → @f43a0e5ff2bd294095638e18286ca9a3d1956744, shivammathur/setup-php@v2 → @f3e473d116dcccaddc5834248c87452386958240, actions/cache@v3 → @6f8efc29b200d32929f49075959781ed54ec270c, codecov/codecov-action@v3 → @ab904c41d6ece82784817410c45d8b8c02684457
All original version tags preserved as inline comments.

