<!-- markdownlint-disable -->

# Hardening Report: Azure--k8s-create-secret/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **Azure--k8s-create-secret/v6.0.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### hardcoded-credentials (severity: high)

Literal plaintext credentials are hardcoded in the integration test workflow. The value 'test-Pass1' is assigned directly to `container-registry-password:` (line 55) and also passed as `--docker-password=test-Pass1` in a kubectl run command (line 60). These are not GitHub Actions secret expressions and match the hardcoded-credentials pattern.

Locations:

- `.github/workflows/integration-tests.yml:55`
- `.github/workflows/integration-tests.yml:60`

### unpinned-uses (severity: high)

The workflow uses an unpinned branch ref `@main` for an external reusable workflow: `uses: OliverMKing/javascript-release-workflow/.github/workflows/tag-and-release.yml@main`. A branch ref is mutable and can be updated to point to arbitrary code, enabling supply-chain attacks. It must be pinned to a full 40-character commit SHA.

Locations:

- `.github/workflows/tag-and-draft.yml:9`

### unpinned-uses (severity: high)

The workflow uses an unpinned tag ref `@v1` for an external reusable workflow: `uses: Azure/action-release-workflows/.github/workflows/release_js_project.yaml@v1`. A tag ref is mutable and can be force-pushed to point to arbitrary code, enabling supply-chain attacks. It must be pinned to a full 40-character commit SHA.

Locations:

- `.github/workflows/release-pr.yml:14`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/defaultLabels.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/integration-tests.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/prettify-code.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/tag-and-draft.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), violating the principle of least privilege.

Locations:

- `.github/workflows/unit-tests.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** hardcoded-credentials, unpinned-uses, missing-permissions

**Notes:**

Fixed all 8 findings across 6 workflow files:

1. integration-tests.yml: Replaced hardcoded 'test-Pass1' password (lines 55, 60) with ${{ secrets.TEST_REGISTRY_PASSWORD }} secret reference. Added `permissions: contents: read` top-level block.

2. tag-and-draft.yml: Pinned OliverMKing/javascript-release-workflow @main to full SHA a2f171c6ca04fea2de31f4dbb6dc576140395512. Added `permissions: {}` top-level block.

3. release-pr.yml: Pinned Azure/action-release-workflows @v1 to full SHA 3c677ba5ab58f5c5c1a6f0cfb176b333b1f27405. (Job already had permissions block.)

4. defaultLabels.yml: Added `permissions: issues: write` and `pull-requests: write` (minimum needed for actions/stale to label and manage issues/PRs).

5. prettify-code.yml: Added `permissions: contents: read` top-level block.

6. unit-tests.yml: Added `permissions: contents: read` top-level block.

