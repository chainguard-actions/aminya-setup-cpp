<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.10.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aminya--setup-cpp/v1.10.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Update Dist' run: block directly interpolates `${{ github.ref_name }}` into a shell command string: `if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]]; then`. An attacker who controls the ref name (e.g. via a specially crafted tag or branch name) could inject arbitrary shell commands. The value should be passed via an env: variable and double-quoted instead.

Locations:

- `.github/workflows/CI.yml:57`

### script-injection (severity: high)

Rule (a): The 'Define Platform Suffix' run: block directly interpolates `${{ matrix.platform }}` into a shell command string: `if [[ "${{ matrix.platform }}" == "linux/amd64" ]]; then`. Any expression interpolated directly inside a run: block is a script-injection risk. The value should be passed via an env: variable and double-quoted instead.

Locations:

- `.github/workflows/CI.yml:220`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and none of its jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release) define job-level `permissions:` blocks. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal permissions block (e.g. `contents: read`) should be added.

Locations:

- `.github/workflows/CI.yml:1`

### unpinned-uses (severity: high)

All `uses:` references in the workflow use mutable version tags instead of immutable 40-character commit SHA hashes, making the workflow vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: actions/checkout@v7, actions/setup-node@v7, oven-sh/setup-bun@v2, actions/upload-artifact@v7, actions/download-artifact@v8, actions/cache@v6, rharkor/caching-for-turbo@v2.5.1, docker/login-action@v4, docker/build-push-action@v7, peter-evans/dockerhub-description@v5, docker/setup-buildx-action@v4, Noelware/docker-manifest-action@0.4.3, softprops/action-gh-release@v3.

Locations:

- `.github/workflows/CI.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all four findings in CI.yml:
1. script-injection (line 57, 'Update Dist'): Moved `${{ github.ref_name }}` into env var `REF_NAME` and used `"$REF_NAME"` in the shell script.
2. script-injection (line 220, 'Define Platform Suffix'): Moved `${{ matrix.platform }}` into env var `MATRIX_PLATFORM` and used `"$MATRIX_PLATFORM"` in the shell script.
3. missing-permissions: Added top-level `permissions: contents: read` and job-level permissions blocks (Build: contents: write; BuildExecutable/Test/Docker/Docker-Manifest: contents: read; Release: contents: write).
4. unpinned-uses: Pinned all 13 action references to full commit SHAs: actions/checkout@3d3c42e5..., actions/setup-node@82076278..., oven-sh/setup-bun@0c5077e5..., actions/upload-artifact@043fb46d..., actions/download-artifact@3e5f45b2..., actions/cache@55cc8345..., rharkor/caching-for-turbo@2238fae6..., docker/login-action@dbcb8138..., docker/build-push-action@53b7df96..., peter-evans/dockerhub-description@1b9a80c0..., docker/setup-buildx-action@37fe6310..., Noelware/docker-manifest-action@b33ab348..., softprops/action-gh-release@efb35369...

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed 8 script injection locations across 4 pairs of steps in the Docker job of .github/workflows/CI.yml. Each 'Tag latest locally' and 'Push latest to Docker Hub' step (for base, llvm, gcc, and mingw image variants) was updated to move ${{ matrix.container.image }}, ${{ matrix.container.tag }}, ${{ env.version }}, and ${{ steps.platform.outputs.suffix }} expressions into step-level env: blocks (as CONTAINER_IMAGE, CONTAINER_TAG, VERSION, PLATFORM_SUFFIX). The shell commands now reference these as plain environment variables, preventing template-engine injection before the shell processes the commands.

