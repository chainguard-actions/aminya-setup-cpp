<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.10.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aminya--setup-cpp/v1.10.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All 42 `uses:` references in CI.yml use mutable version tags instead of pinned 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: `actions/checkout@v7`, `actions/setup-node@v7`, `oven-sh/setup-bun@v2`, `actions/upload-artifact@v7`, `actions/download-artifact@v8`, `actions/cache@v6`, `rharkor/caching-for-turbo@v2.5.1`, `docker/login-action@v4`, `docker/build-push-action@v7`, `peter-evans/dockerhub-description@v5`, `docker/setup-buildx-action@v4`, `Noelware/docker-manifest-action@0.4.3`, `softprops/action-gh-release@v3`.

Locations:

- `.github/workflows/CI.yml:22`
- `.github/workflows/CI.yml:27`
- `.github/workflows/CI.yml:31`
- `.github/workflows/CI.yml:57`
- `.github/workflows/CI.yml:72`
- `.github/workflows/CI.yml:78`
- `.github/workflows/CI.yml:83`
- `.github/workflows/CI.yml:88`
- `.github/workflows/CI.yml:93`
- `.github/workflows/CI.yml:101`
- `.github/workflows/CI.yml:104`

### missing-permissions (severity: medium)

The workflow file CI.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release). Without explicit permissions, the workflow inherits the default repository permissions, which may be overly broad (e.g., write access to contents). Minimal permissions should be declared explicitly.

Locations:

- `.github/workflows/CI.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in CI.yml directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell commands, violating rule (a). This allows expression values to be interpreted as shell code before the shell ever sees them.

1. 'Update Dist' step (Build job): `if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]]` — `github.ref_name` is attacker-controllable via branch/tag names.

2. 'Define Platform Suffix' step (Docker job): `if [[ "${{ matrix.platform }}" == "linux/amd64" ]]` — matrix values flow through YAML template substitution.

3. Multiple 'Tag latest locally' and 'docker push' steps: `docker tag aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}-${{ env.version }}-${{ steps.platform.outputs.suffix }} ...` — matrix context and step outputs are interpolated directly into shell commands.

4. 'Smoke Test Modern Bundle' and 'Smoke Test Legacy Bundle' steps use `${{ matrix.os }}` via env but the docker steps use matrix values directly in run blocks.

Locations:

- `.github/workflows/CI.yml:44`
- `.github/workflows/CI.yml:193`
- `.github/workflows/CI.yml:222`
- `.github/workflows/CI.yml:258`
- `.github/workflows/CI.yml:295`
- `.github/workflows/CI.yml:338`
- `.github/workflows/CI.yml:375`
- `.github/workflows/CI.yml:415`
- `.github/workflows/CI.yml:452`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 42 unpinned uses: references by resolving each action tag to its full 40-character commit SHA (e.g., actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 # v7). Added top-level 'permissions: contents: read' and job-level permissions blocks: Build and Release get 'contents: write' (needed for git push and release creation), all other jobs get 'contents: read'. Fixed script injection in all run: blocks by moving ${{ }} expressions to env: blocks: github.ref_name → REF_NAME in Update Dist step; matrix.platform → MATRIX_PLATFORM in Define Platform Suffix step; matrix.container.image/tag, steps.platform.outputs.suffix, env.version → CONTAINER_IMAGE/CONTAINER_TAG/PLATFORM_SUFFIX/APP_VERSION in all Tag latest locally and Push latest to Docker Hub steps across the Docker job.

