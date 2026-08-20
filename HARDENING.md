<!-- markdownlint-disable -->

# Hardening Report: peter-evans--dockerhub-description/v5.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--dockerhub-description/v5.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags (e.g. @v3, @v4, @v5, @v7) instead of pinned full 40-character commit SHAs. This exposes the workflow to supply-chain attacks if a tag is moved to a malicious commit. Affected references include: actions/checkout@v5, actions/setup-node@v5, actions/upload-artifact@v4, actions/download-artifact@v5, peter-evans/create-pull-request@v7, peter-evans/enable-pull-request-automerge@v3.

Locations:

- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:27`
- `.github/workflows/ci.yml:28`
- `.github/workflows/ci.yml:35`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:68`
- `.github/workflows/ci.yml:69`
- `.github/workflows/ci.yml:76`
- `.github/workflows/automerge-dependabot.yml:10`
- `.github/workflows/dockerhub-description.yml:14`
- `.github/workflows/publish-docker.yml:13`
- `.github/workflows/update-major-version.yml:17`

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings, allowing injection of arbitrary shell commands. In publish-docker.yml, `${{ secrets.DOCKERHUB_PASSWORD }}` and `${{ secrets.DOCKERHUB_USERNAME }}` are interpolated directly in a run: block (line 17), and `${{ github.ref }}` is interpolated directly in a run: block (lines 22-23). In update-major-version.yml, `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` — both user-supplied workflow_dispatch inputs — are interpolated directly in run: blocks (lines 23-24), enabling direct command injection by any user who can trigger the workflow.

Locations:

- `.github/workflows/publish-docker.yml:17`
- `.github/workflows/publish-docker.yml:22`
- `.github/workflows/publish-docker.yml:23`
- `.github/workflows/update-major-version.yml:23`
- `.github/workflows/update-major-version.yml:24`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad, e.g. write access to contents). Affected files: automerge-dependabot.yml, dockerhub-description.yml, publish-docker.yml, update-major-version.yml.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/dockerhub-description.yml:1`
- `.github/workflows/publish-docker.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 5 workflow files:

1. unpinned-uses (ci.yml, automerge-dependabot.yml, dockerhub-description.yml, publish-docker.yml, update-major-version.yml): Pinned all 6 action references to full 40-character commit SHAs with tag comments for readability.

2. script-injection (publish-docker.yml, update-major-version.yml): Moved all ${{ }} expressions out of run: shell strings into step-level env: blocks. In publish-docker.yml, DOCKERHUB_PASSWORD, DOCKERHUB_USERNAME, and github.ref are now env vars. In update-major-version.yml, github.event.inputs.main_version and github.event.inputs.target are now env vars referenced as $MAIN_VERSION and $TARGET.

3. missing-permissions (automerge-dependabot.yml, dockerhub-description.yml, publish-docker.yml, update-major-version.yml): Added top-level permissions blocks with minimal required permissions: automerge-dependabot gets pull-requests:write + contents:write; dockerhub-description gets contents:read; publish-docker gets contents:read; update-major-version gets contents:write.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all unquoted variable expansions in .github/workflows/publish-docker.yml. Added double-quotes around every occurrence of $VERSION and $IMAGE_NAME in the 'Push image to Docker Hub' step's run block: (1) `echo $VERSION` in the strip-v-prefix command, (2) `--tag $IMAGE_NAME` in docker build, (3) both arguments to `docker tag` for the full version, minor version, and major version tags, (4) the `echo $VERSION` inputs to the MINOR_VERSION and MAJOR_VERSION sed commands, and (5) `docker push $IMAGE_NAME`. This prevents shell word-splitting and glob expansion on attacker-controlled tag name values.

