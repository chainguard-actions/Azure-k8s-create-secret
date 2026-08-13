<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v5.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v5.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags or branch names instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks.

.github/workflows/integration-tests.yml:
  - `uses: medyagh/setup-minikube@latest` (line 17, line ~57)
  - `uses: actions/checkout@v1` (line 23, line ~63)

.github/workflows/unit-tests.yml:
  - `uses: actions/checkout@v1` (line 8)

.github/workflows/defaultLabels.yml:
  - `uses: actions/stale@v3` (lines 16, 27)

.github/workflows/prettify-code.yml:
  - `uses: actions/checkout@v2` (line 11)
  - `uses: actionsx/prettier@v2` (line 14)

.github/workflows/release-pr.yml:
  - `uses: Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1` (line 13)

.github/workflows/tag-and-draft.yml:
  - `uses: OliverMKing/javascript-release-workflow/.github/workflows/tag-and-release.yml@main` (line 7)

Locations:

- `.github/workflows/integration-tests.yml:17`
- `.github/workflows/integration-tests.yml:23`
- `.github/workflows/unit-tests.yml:8`
- `.github/workflows/defaultLabels.yml:16`
- `.github/workflows/prettify-code.yml:11`
- `.github/workflows/prettify-code.yml:14`
- `.github/workflows/release-pr.yml:13`
- `.github/workflows/tag-and-draft.yml:7`

### permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any job, granting the default (potentially broad) token permissions:

- integration-tests.yml: two jobs, neither has permissions
- unit-tests.yml: one job, no permissions
- defaultLabels.yml: one job, no permissions
- prettify-code.yml: one job, no permissions
- tag-and-draft.yml: one job, no permissions

(release-pr.yml is compliant — it has job-level permissions.)

Locations:

- `.github/workflows/integration-tests.yml:1`
- `.github/workflows/unit-tests.yml:1`
- `.github/workflows/defaultLabels.yml:1`
- `.github/workflows/prettify-code.yml:1`
- `.github/workflows/tag-and-draft.yml:1`

### script-injection (severity: high)

In .github/workflows/integration-tests.yml, the env var `PR_BASE_REF` is populated from the attacker-controllable value `${{ github.event.pull_request.base.ref }}` and then expanded **unquoted** inside `run:` shell scripts (rule b). An attacker controlling the base ref of a pull request could inject shell metacharacters.

Violating lines in job `aks-minikube-integration-tests` (step `action-npm-build`):
  - `echo $PR_BASE_REF`  (unquoted)
  - `if [[ $PR_BASE_REF != releases/* ]]; then`  (unquoted)

Violating lines in job `aks-minikube-docker-registry-tests` (step `action-npm-build`):
  - `echo $PR_BASE_REF`  (unquoted)
  - `if [[ $PR_BASE_REF != releases/* ]]; then`  (unquoted)

Fix: quote all expansions as `"$PR_BASE_REF"`.

Locations:

- `.github/workflows/integration-tests.yml:27`
- `.github/workflows/integration-tests.yml:28`

### hardcoded-credentials (severity: high)

The file .github/workflows/integration-tests.yml contains hardcoded literal credential values (even if intended as test credentials, they match the hardcoded-credentials pattern and should be stored as secrets):

1. `container-registry-password: 'test-Pass1'` — literal password passed to the action's `with:` block.
2. `--docker-password=test-Pass1` — literal password passed directly on the kubectl command line, visible in runner logs.

Locations:

- `.github/workflows/integration-tests.yml:75`
- `.github/workflows/integration-tests.yml:80`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across six workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with original tag as comment: medyagh/setup-minikube@latest→e9e035a, actions/checkout@v1→50fbc62, actions/checkout@v2→ee0669b, actions/stale@v3→98ed4cb, actionsx/prettier@v2→8b6d14b, Azure/action-release-workflows@v1→3c677ba, OliverMKing/javascript-release-workflow@main→a2f171c.

2. **permissions**: Added top-level `permissions: {}` to integration-tests.yml, unit-tests.yml, defaultLabels.yml, prettify-code.yml, and tag-and-draft.yml. Added minimal job-level permissions where needed (issues+pull-requests write for stale, contents read for prettier, contents write for tag-and-release).

3. **script-injection**: Quoted all `$PR_BASE_REF` expansions in both jobs of integration-tests.yml (`echo "$PR_BASE_REF"` and `if [[ "$PR_BASE_REF" != releases/* ]]`).

4. **hardcoded-credentials**: Replaced literal `test-Pass1` password with `${{ secrets.CONTAINER_REGISTRY_PASSWORD }}` in the action `with:` block, and moved it to an `env:` variable (`DOCKER_PASSWORD`) for the kubectl command to avoid shell injection.

