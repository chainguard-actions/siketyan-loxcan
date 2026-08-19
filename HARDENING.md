<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.10.1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The composite action's run: block in action.yml directly interpolates multiple ${{ }} expressions into shell commands (sub-rule a). This allows an attacker who controls input values to inject arbitrary shell commands. Affected interpolations include: `pushd '${{ github.action_path }}'`, `if [ "${{ inputs.report_enabled }}" = "true" ]`, `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"`, `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"`, `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"`, `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"`, `BRANCH_BASE="origin/${{ inputs.base }}"`, `BRANCH_HEAD="${{ github.sha }}"`, and `${{ github.action_path }}/bin/loxcan ...`. All ${{ }} expressions should be moved to env: vars and the env vars used in the run: block instead.

Locations:

- `action.yml:38`

### unpinned-uses (severity: high)

All uses: references across the workflow files use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised. Unpinned references: actions/checkout@v4 (.github/workflows/action.yml), actions/checkout@v4 (.github/workflows/docker.yml), docker/setup-buildx-action@v3 (.github/workflows/docker.yml), docker/login-action@v3 (.github/workflows/docker.yml), docker/build-push-action@v5 (.github/workflows/docker.yml), actions/checkout@v4 (.github/workflows/php.yml), shivammathur/setup-php@v2 (.github/workflows/php.yml), actions/cache@v3 (.github/workflows/php.yml), codecov/codecov-action@v3 (.github/workflows/php.yml).

Locations:

- `.github/workflows/action.yml:16`
- `.github/workflows/docker.yml:14`
- `.github/workflows/docker.yml:18`
- `.github/workflows/docker.yml:23`
- `.github/workflows/docker.yml:35`
- `.github/workflows/php.yml:18`
- `.github/workflows/php.yml:22`
- `.github/workflows/php.yml:31`
- `.github/workflows/php.yml:52`

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

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses

**Notes:**

Fixed script injection in action.yml by moving all ${{ }} expressions (github.action_path, inputs.report_enabled, inputs.owner, inputs.repo, inputs.issue_number, inputs.token, inputs.base, github.sha) into an env: block and referencing them as plain shell variables. Pinned all 9 unpinned action references across .github/workflows/action.yml, .github/workflows/docker.yml, and .github/workflows/php.yml to their full 40-character SHA commit hashes with version tag comments for readability.

