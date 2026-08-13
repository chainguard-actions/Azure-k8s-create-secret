<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v5.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference Actions and reusable workflows using mutable tags/branches instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks:
- defaultLabels.yml: `actions/stale@v3` (×2)
- integration-tests.yml: `medyagh/setup-minikube@latest` (×2), `actions/checkout@v1` (×2)
- prettify-code.yml: `actions/checkout@v2`, `actionsx/prettier@v2`
- release-pr.yml: `Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1`
- tag-and-draft.yml: `OliverMKing/javascript-release-workflow/.github/workflows/tag-and-release.yml@main`
- unit-tests.yml: `actions/checkout@v1`

Locations:

- `.github/workflows/defaultLabels.yml:16`
- `.github/workflows/defaultLabels.yml:27`
- `.github/workflows/integration-tests.yml:15`
- `.github/workflows/integration-tests.yml:18`
- `.github/workflows/integration-tests.yml:55`
- `.github/workflows/integration-tests.yml:58`
- `.github/workflows/prettify-code.yml:11`
- `.github/workflows/prettify-code.yml:14`
- `.github/workflows/release-pr.yml:11`
- `.github/workflows/tag-and-draft.yml:8`
- `.github/workflows/unit-tests.yml:12`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job, meaning they run with the default (potentially broad) token permissions:
- defaultLabels.yml: no permissions declared
- integration-tests.yml: no permissions declared
- prettify-code.yml: no permissions declared
- tag-and-draft.yml: no permissions declared
- unit-tests.yml: no permissions declared

Locations:

- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/integration-tests.yml:1`
- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/tag-and-draft.yml:1`
- `.github/workflows/unit-tests.yml:1`

### script-injection (severity: high)

Sub-rule (b): In integration-tests.yml, the env var `PR_BASE_REF` is populated from `${{ github.event.pull_request.base.ref }}` (attacker-controllable via a PR) and then used **unquoted** in two `run:` blocks. Unquoted shell expansion allows shell metacharacters in the branch name to be interpreted by the shell, enabling command injection.

Offending lines:
  `echo $PR_BASE_REF`
  `if [[ $PR_BASE_REF != releases/* ]]; then`

These appear in both the `aks-minikube-integration-tests` job and the `aks-minikube-docker-registry-tests` job. The value should be double-quoted: `"$PR_BASE_REF"`.

Locations:

- `.github/workflows/integration-tests.yml:24`
- `.github/workflows/integration-tests.yml:25`
- `.github/workflows/integration-tests.yml:64`
- `.github/workflows/integration-tests.yml:65`

### hardcoded-credentials (severity: high)

integration-tests.yml contains literal hardcoded test credentials. The value `test-Pass1` is assigned to `container-registry-password:` in a `with:` block and also passed as `--docker-password=test-Pass1` in a `run:` block. Even though these appear to be test/dummy values, they are literal non-expression credential values matching the hardcoded-credentials pattern.

Locations:

- `.github/workflows/integration-tests.yml:76`
- `.github/workflows/integration-tests.yml:83`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all 4 findings across 6 workflow files:

1. unpinned-uses: Pinned all 7 unique action references to full 40-char SHAs using lookup_action_sha. Preserved original tag/branch in trailing comments for readability.

2. missing-permissions: Added `permissions: {}` top-level block to defaultLabels.yml, integration-tests.yml, prettify-code.yml, tag-and-draft.yml, and unit-tests.yml. release-pr.yml already had job-level permissions and was not in the missing-permissions list.

3. script-injection: In integration-tests.yml, both jobs had `echo $PR_BASE_REF` and `if [[ $PR_BASE_REF != releases/* ]]` with unquoted variable expansion. Fixed by double-quoting: `echo "$PR_BASE_REF"` and `if [[ "$PR_BASE_REF" != releases/* ]]`.

4. hardcoded-credentials: Replaced literal `test-Pass1` password in integration-tests.yml with `${{ secrets.TEST_REGISTRY_PASSWORD }}` in the action `with:` block, and used an `env:` block with `REGISTRY_PASSWORD: ${{ secrets.TEST_REGISTRY_PASSWORD }}` for the kubectl run step, referencing it as `"$REGISTRY_PASSWORD"` in the shell command.

