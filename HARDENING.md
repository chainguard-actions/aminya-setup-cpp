<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aminya--setup-cpp/v1.8.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All uses: references in .github/workflows/CI.yml use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflow vulnerable to supply-chain attacks. Failing references include: actions/checkout@v6, actions/setup-node@v6, oven-sh/setup-bun@v2, actions/cache@v5, rharkor/caching-for-turbo@v1.8, actions/upload-artifact@v4, actions/download-artifact@v5, docker/login-action@v3, docker/build-push-action@v6, peter-evans/dockerhub-description@v5, docker/setup-buildx-action@v3, Noelware/docker-manifest-action@0.4.3, softprops/action-gh-release@v2.

Locations:

- `.github/workflows/CI.yml:27`

### permissions (severity: medium)

The workflow file .github/workflows/CI.yml has no top-level permissions: key and no job-level permissions: block on any of its jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release). The workflow runs with default overly-broad repository permissions.

Locations:

- `.github/workflows/CI.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ ... }} expressions into shell commands (sub-rule a). (1) Update Dist step: if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]] — github.ref_name is attacker-controllable via tag/branch names. (2) Define Platform Suffix step: if [[ "${{ matrix.platform }}" == "linux/amd64" ]] — matrix values flow through YAML template substitution before the shell sees them. (3) Multiple Tag latest locally steps use ${{ matrix.container.image }}, ${{ matrix.container.tag }}, ${{ env.version }}, ${{ steps.platform.outputs.suffix }} directly in docker tag commands. (4) Multiple Push latest to Docker Hub steps use ${{ matrix.container.image }} and ${{ matrix.container.tag }} directly in docker push commands. All these expressions should be moved to env: variables and referenced as double-quoted shell variables.

Locations:

- `.github/workflows/CI.yml:52`
- `.github/workflows/CI.yml:215`
- `.github/workflows/CI.yml:247`
- `.github/workflows/CI.yml:261`
- `.github/workflows/CI.yml:296`
- `.github/workflows/CI.yml:330`
- `.github/workflows/CI.yml:365`
- `.github/workflows/CI.yml:399`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection

**Notes:**

Fixed all three findings in .github/workflows/CI.yml:

1. unpinned-uses: Pinned all 13 action references to full 40-character SHA commit hashes (actions/checkout, actions/setup-node, oven-sh/setup-bun, actions/cache, rharkor/caching-for-turbo, actions/upload-artifact, actions/download-artifact, docker/login-action, docker/build-push-action, peter-evans/dockerhub-description, docker/setup-buildx-action, Noelware/docker-manifest-action, softprops/action-gh-release). Original tags preserved as comments.

2. permissions: Added top-level 'permissions: {}' to deny all by default, then added minimal job-level permissions: Build and Release jobs get 'contents: write' (needed for git push and release creation), all other jobs get 'contents: read'.

3. script-injection: Moved all ${{ }} expressions out of run: shell commands into env: blocks. Fixed: github.ref_name in Update Dist step, matrix.platform in Define Platform Suffix step, and matrix.container.image/tag/env.version/steps.platform.outputs.suffix in all Tag latest locally and Push latest to Docker Hub steps across the Docker job.

