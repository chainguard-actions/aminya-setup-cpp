<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.9.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aminya--setup-cpp/v1.9.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All action references in .github/workflows/CI.yml use mutable tags or version strings instead of pinned 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks if any upstream action is compromised or a tag is moved. Failing references include: actions/checkout@v7, actions/setup-node@v6, oven-sh/setup-bun@v2, actions/upload-artifact@v6, actions/download-artifact@v7, actions/cache@v6, rharkor/caching-for-turbo@v2.5.1, docker/login-action@v4, docker/build-push-action@v7, peter-evans/dockerhub-description@v5, docker/setup-buildx-action@v4, Noelware/docker-manifest-action@0.4.3, softprops/action-gh-release@v2.

Locations:

- `.github/workflows/CI.yml:27`
- `.github/workflows/CI.yml:32`
- `.github/workflows/CI.yml:37`
- `.github/workflows/CI.yml:68`
- `.github/workflows/CI.yml:75`
- `.github/workflows/CI.yml:80`
- `.github/workflows/CI.yml:85`
- `.github/workflows/CI.yml:96`
- `.github/workflows/CI.yml:101`

### missing-permissions (severity: medium)

The workflow file .github/workflows/CI.yml has no top-level `permissions:` key and none of the jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release) define job-level `permissions:` blocks. The workflow therefore runs with GitHub's default permissions, which may include write access to repository contents and other resources beyond what is needed.

Locations:

- `.github/workflows/CI.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks in CI.yml directly interpolate `${{ }}` expressions into shell commands (rule a), allowing an attacker to inject arbitrary shell commands via matrix values or GitHub context values.

1. 'Update Dist' step (Build job): `if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]];` — github.ref_name is interpolated directly into a shell conditional.

2. 'Define Platform Suffix' step (Docker job): `if [[ "${{ matrix.platform }}" == "linux/amd64" ]];` — matrix.platform is interpolated directly into a shell conditional.

3. Multiple 'Tag latest locally' steps (Docker job): `docker tag aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}-${{ env.version }}-${{ steps.platform.outputs.suffix }} ...` — matrix and env context values interpolated directly into docker commands.

4. Multiple 'Push latest to Docker Hub' steps (Docker job): `docker push aminya/${{ matrix.container.image }}:latest` and `docker push aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}` — matrix values interpolated directly into docker commands.

All of these should use environment variables set in an `env:` block and referenced as `"$ENV_VAR"` in the shell script instead.

Locations:

- `.github/workflows/CI.yml:57`
- `.github/workflows/CI.yml:211`
- `.github/workflows/CI.yml:231`
- `.github/workflows/CI.yml:238`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all three findings in CI.yml: (1) Pinned all 13 action references to full SHA hashes with tag comments; (2) Added top-level `permissions: {}` and job-level permissions blocks (contents: write for Build/Release, contents: read for other jobs); (3) Moved all ${{ }} expressions from run: shell scripts into env: blocks - github.ref_name → GIT_REF_NAME in Update Dist step, matrix.platform → MATRIX_PLATFORM in Define Platform Suffix step, and all docker tag/push commands now use CONTAINER_IMAGE, CONTAINER_TAG, PLATFORM_SUFFIX, APP_VERSION env vars.

