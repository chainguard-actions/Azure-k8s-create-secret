<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v6.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### missing-permissions (severity: medium)

The workflow file 'defaultLabels.yml' has no top-level 'permissions:' key and none of its jobs define a 'permissions:' block. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/defaultLabels.yml:9`

### missing-permissions (severity: medium)

The workflow file 'integration-tests.yml' has no top-level 'permissions:' key and none of its jobs (aks-minikube-integration-tests, aks-minikube-docker-registry-tests) define a 'permissions:' block. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/integration-tests.yml:5`

### missing-permissions (severity: medium)

The workflow file 'prettify-code.yml' has no top-level 'permissions:' key and its only job ('prettier') does not define a 'permissions:' block. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/prettify-code.yml:1`

### missing-permissions (severity: medium)

The workflow file 'unit-tests.yml' has no top-level 'permissions:' key and its only job ('build') does not define a 'permissions:' block. Without explicit permissions, the workflow inherits the default (potentially broad) repository permissions.

Locations:

- `.github/workflows/unit-tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** missing-permissions

**Notes:**

Added top-level 'permissions:' blocks to all four workflow files: (1) defaultLabels.yml — granted 'issues: write' and 'pull-requests: write' required by actions/stale to manage stale labels; (2) integration-tests.yml — granted 'contents: read' for code checkout; (3) prettify-code.yml — granted 'contents: read' for code checkout; (4) unit-tests.yml — granted 'contents: read' for code checkout. All permissions follow the principle of least privilege.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, hardcoded-credentials

**Notes:**

1. script-injection (action.yml lines 33-34): Moved `${{ inputs.pr-base-ref }}` into an `env:` block as `PR_BASE_REF` and updated the shell script to use `"$PR_BASE_REF"` safely. 2. unpinned-uses (action.yml line 24): Pinned `medyagh/setup-minikube@latest` to full SHA `e9e035a86bbc3caea26a450bd4dbf9d0c453682e # latest`. 3. hardcoded-credentials (integration-tests.yml line 56): Replaced literal password `'test-Pass1'` with `${{ secrets.TEST_REGISTRY_PASSWORD }}`.

### Iteration 3

**Fixes applied:** hardcoded-credentials

**Notes:**

Replaced the hardcoded password `test-Pass1` in the kubectl create secret command at `.github/workflows/integration-tests.yml` line 62 with the GitHub Actions secret expression `${{ secrets.TEST_KUBECTL_PASSWORD }}`. The credential is now sourced from GitHub Actions secrets rather than being committed as a literal value in the workflow file.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/.github/workflows/integration-tests.yml at line 58. Moved `${{ secrets.TEST_KUBECTL_PASSWORD }}` out of the `run:` shell command into an `env:` block as `KUBECTL_PASSWORD`, then referenced it as `"$KUBECTL_PASSWORD"` in the kubectl command. This prevents shell metacharacters in the secret value from being interpreted by the shell.

