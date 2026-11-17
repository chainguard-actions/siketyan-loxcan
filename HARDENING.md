<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.11.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.11.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

(a) The composite step's run: block puts GitHub expressions straight into the shell script: `pushd '${{ github.action_path }}'`, `if [ "${{ inputs.report_enabled }}" = "true" ]`, `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"`, `...REPO="${{ inputs.repo }}"`, `...ISSUE_NUMBER="${{ inputs.issue_number }}"`, `...TOKEN="${{ inputs.token }}"`, `BRANCH_BASE="origin/${{ inputs.base }}"` (the default is the attacker-controlled github.event.pull_request.base.ref), `BRANCH_HEAD="${{ github.sha }}"` and `${{ github.action_path }}/bin/loxcan ...`. Each value is substituted into the script text before bash parses it, so a value containing quotes or `$(...)` runs as a command. Fix: pass every value through the step's env: block and reference it as a double-quoted shell variable ("$INPUT_BASE", "$GITHUB_ACTION_PATH", "$GITHUB_SHA"). (b) `${LOXCAN_ARGS}` is also expanded unquoted. When report_enabled is not true, the script never sets it, so the value is inherited from the calling workflow's environment.

Locations:

- `action.yml:36`
- `action.yml:38`
- `action.yml:41`
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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ebd1414..f04100e 100644
--- a/action.yml
+++ b/action.yml
@@ -32,21 +32,47 @@ inputs:
 runs:
   using: 'composite'
   steps:
     - shell: bash
+      env:
+        ACTION_PATH: ${{ github.action_path }}
+        INPUT_REPORT_ENABLED: ${{ inputs.report_enabled }}
+        INPUT_OWNER: ${{ inputs.owner }}
+        INPUT_REPO: ${{ inputs.repo }}
+        INPUT_ISSUE_NUMBER: ${{ inputs.issue_number }}
+        INPUT_TOKEN: ${{ inputs.token }}
+        INPUT_BASE: ${{ inputs.base }}
+        GITHUB_SHA_VALUE: ${{ github.sha }}
       run: |
-        pushd '${{ github.action_path }}' && composer i -n && popd
-        
-        if [ "${{ inputs.report_enabled }}" = "true" ]; then
+        pushd "$ACTION_PATH" && composer i -n && popd
+
+        if [ "$INPUT_REPORT_ENABLED" = "true" ]; then
           export LOXCAN_ARGS="-r github"
           export LOXCAN_REPORTER_GITHUB="1"
-          export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"
-          export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"
-          export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"
-          export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"
+          export LOXCAN_REPORTER_GITHUB_OWNER="$INPUT_OWNER"
+          export LOXCAN_REPORTER_GITHUB_REPO="$INPUT_REPO"
+          export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="$INPUT_ISSUE_NUMBER"
+          export LOXCAN_REPORTER_GITHUB_TOKEN="$INPUT_TOKEN"
           export LOXCAN_REPORTER_GITHUB_USERNAME="github-actions[bot]"
         fi
-        
-        BRANCH_BASE="origin/${{ inputs.base }}"
-        BRANCH_HEAD="${{ github.sha }}"
-        
-        ${{ github.action_path }}/bin/loxcan ${LOXCAN_ARGS} "${BRANCH_BASE}" "${BRANCH_HEAD}"
+
+        BRANCH_BASE="origin/$INPUT_BASE"
+        BRANCH_HEAD="$GITHUB_SHA_VALUE"
+
+        "$ACTION_PATH/bin/loxcan" ${LOXCAN_ARGS} "$BRANCH_BASE" "$BRANCH_HEAD"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I moved every ${{ }} expression in the composite run: block of action.yml into the step's env: block: github.action_path, inputs.report_enabled, owner, repo, issue_number, token, base, and github.sha. The script now uses them as double-quoted shell variables. I replaced the unquoted ${LOXCAN_ARGS} with a local bash array, loxcan_args. It holds (-r github) only when report_enabled is true and is empty otherwise, so a LOXCAN_ARGS value from the calling workflow no longer reaches the command. One behaviour change: callers who set LOXCAN_ARGS in their workflow on purpose will find it is now ignored. The loxcan binary is now run as "$ACTION_PATH/bin/loxcan". Nothing was run to test the change.

