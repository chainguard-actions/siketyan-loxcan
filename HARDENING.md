<!-- markdownlint-disable -->

# Hardening Report: siketyan--loxcan/v0.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **siketyan--loxcan/v0.10.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the composite step's run: block in action.yml, GitHub Actions expressions are pasted straight into the shell script, so the runner substitutes their values before bash parses anything. The affected lines are: `pushd '${{ github.action_path }}'`, `if [ "${{ inputs.report_enabled }}" = "true" ]`, `export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"`, `export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"`, `export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"`, `export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"`, `BRANCH_BASE="origin/${{ inputs.base }}"`, `BRANCH_HEAD="${{ github.sha }}"` and `${{ github.action_path }}/bin/loxcan ...`. The default for inputs.base is github.event.pull_request.base.ref, and a calling workflow can set any of these inputs, so a crafted value can break out of the quotes and run commands. Rule (b): `${LOXCAN_ARGS}` is expanded unquoted. The script sets it only when reporting is enabled. Otherwise it keeps whatever value the calling workflow's environment gave it, so it holds workflow-controlled data. Fix: pass each value through the step's env: block and expand it double-quoted ("$VAR"). Use "$GITHUB_ACTION_PATH" and "$GITHUB_SHA" instead of the github.* expressions. Set LOXCAN_ARGS explicitly in both branches, or build it as an array.

Locations:

- `action.yml:36`
- `action.yml:38`
- `action.yml:41`
- `action.yml:44`
- `action.yml:49`
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

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index ebd1414..aef3246 100644
--- a/action.yml
+++ b/action.yml
@@ -32,21 +32,47 @@ inputs:
 runs:
   using: 'composite'
   steps:
     - shell: bash
+      env:
+        ACTION_PATH: ${{ github.action_path }}
+        REPORT_ENABLED: ${{ inputs.report_enabled }}
+        OWNER: ${{ inputs.owner }}
+        REPO: ${{ inputs.repo }}
+        ISSUE_NUMBER: ${{ inputs.issue_number }}
+        TOKEN: ${{ inputs.token }}
+        BASE: ${{ inputs.base }}
+        GH_SHA: ${{ github.sha }}
       run: |
-        pushd '${{ github.action_path }}' && composer i -n && popd
-        
-        if [ "${{ inputs.report_enabled }}" = "true" ]; then
+        pushd "$ACTION_PATH" && composer i -n && popd
+
+        if [ "$REPORT_ENABLED" = "true" ]; then
           export LOXCAN_ARGS="-r github"
           export LOXCAN_REPORTER_GITHUB="1"
-          export LOXCAN_REPORTER_GITHUB_OWNER="${{ inputs.owner }}"
-          export LOXCAN_REPORTER_GITHUB_REPO="${{ inputs.repo }}"
-          export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="${{ inputs.issue_number }}"
-          export LOXCAN_REPORTER_GITHUB_TOKEN="${{ inputs.token }}"
+          export LOXCAN_REPORTER_GITHUB_OWNER="$OWNER"
+          export LOXCAN_REPORTER_GITHUB_REPO="$REPO"
+          export LOXCAN_REPORTER_GITHUB_ISSUE_NUMBER="$ISSUE_NUMBER"
+          export LOXCAN_REPORTER_GITHUB_TOKEN="$TOKEN"
           export LOXCAN_REPORTER_GITHUB_USERNAME="github-actions[bot]"
         fi
-        
-        BRANCH_BASE="origin/${{ inputs.base }}"
-        BRANCH_HEAD="${{ github.sha }}"
-        
-        ${{ github.action_path }}/bin/loxcan ${LOXCAN_ARGS} "${BRANCH_BASE}" "${BRANCH_HEAD}"
+
+        BRANCH_BASE="origin/$BASE"
+        BRANCH_HEAD="$GH_SHA"
+
+        "$ACTION_PATH/bin/loxcan" ${LOXCAN_ARGS} "$BRANCH_BASE" "$BRANCH_HEAD"

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

Moved all inputs (report_enabled, owner, repo, issue_number, token, base) into the step env: block and expanded them double-quoted. Replaced github.action_path/github.sha expressions with $GITHUB_ACTION_PATH/$GITHUB_SHA. Replaced the unquoted, env-inheritable ${LOXCAN_ARGS} with a locally initialized bash array (empty by default, '-r github' when reporting is enabled).

