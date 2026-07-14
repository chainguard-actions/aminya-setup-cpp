<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.8.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **aminya--setup-cpp/v1.8.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### permissions (severity: medium)

The workflow file .github/workflows/CI.yml has no top-level 'permissions:' key and no job-level 'permissions:' keys on any of its jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release). This grants the default (potentially broad) permissions to all jobs.

Locations:

- `.github/workflows/CI.yml:1`

### unpinned-uses (severity: high)

All 'uses:' references in CI.yml use mutable version tags instead of full 40-character SHA digests, making the workflow vulnerable to supply-chain attacks. Unpinned references include: actions/checkout@v6 (lines 27, 88, 161, 200), actions/setup-node@v6 (lines 32, 93, 168, 178, 191), oven-sh/setup-bun@v2 (lines 37, 98, 173), actions/upload-artifact@v4 (lines 72, 130), actions/download-artifact@v5 (lines 90, 163), actions/cache@v5 (lines 103, 116), rharkor/caching-for-turbo@v1.8 (line 101), docker/login-action@v3 (lines 252, 399), docker/build-push-action@v6 (multiple), peter-evans/dockerhub-description@v5 (multiple), docker/setup-buildx-action@v3 (line 397), Noelware/docker-manifest-action@0.4.3 (lines 410, 420, 430), softprops/action-gh-release@v2 (line 467).

Locations:

- `.github/workflows/CI.yml:27`

### script-injection (severity: high)

Multiple run: blocks in CI.yml directly interpolate ${{ }} expressions into shell commands, violating rule (a):

(1) 'Update Dist' step: `if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]]` — github.ref_name is interpolated directly into a shell conditional. An attacker controlling a branch name could inject shell metacharacters.

(2) 'Define Platform Suffix' step: `if [[ "${{ matrix.platform }}" == "linux/amd64" ]]` — matrix.platform is interpolated directly into a shell conditional.

(3) Multiple 'Tag latest locally' steps: `docker tag aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}-${{ env.version }}-${{ steps.platform.outputs.suffix }} ...` — matrix and env context values interpolated directly into shell commands.

(4) Multiple 'Push latest to Docker Hub' steps: `docker push aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}` — matrix context values interpolated directly into shell commands.

Locations:

- `.github/workflows/CI.yml:57`
- `.github/workflows/CI.yml:246`
- `.github/workflows/CI.yml:259`
- `.github/workflows/CI.yml:265`
- `.github/workflows/CI.yml:306`
- `.github/workflows/CI.yml:312`
- `.github/workflows/CI.yml:341`
- `.github/workflows/CI.yml:347`
- `.github/workflows/CI.yml:376`
- `.github/workflows/CI.yml:382`

## Iteration Notes

### Iteration 1

**Fixes applied:** permissions, unpinned-uses, script-injection

**Notes:**

Fixed all three findings in .github/workflows/CI.yml: (1) Added top-level 'permissions: {}' and job-level permissions blocks to all 6 jobs (Build: contents:write, BuildExecutable/Test/Docker/Docker-Manifest: contents:read, Release: contents:write). (2) Pinned all 13 unpinned action references to full 40-character SHA digests with tag comments: actions/checkout@df4cb1c069e1874edd31b4311f1884172cec0e10 #v6, actions/setup-node@48b55a011bda9f5d6aeb4c2d9c7362e8dae4041e #v6, oven-sh/setup-bun@0c5077e51419868618aeaa5fe8019c62421857d6 #v2, actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 #v4, actions/download-artifact@634f93cb2916e3fdff6788551b99b062d0335ce0 #v5, actions/cache@caa296126883cff596d87d8935842f9db880ef25 #v5, rharkor/caching-for-turbo@a1c4079258ae08389be75b57d4d7a70f23c1c66d #v1.8, docker/login-action@c94ce9fb468520275223c153574b00df6fe4bcc9 #v3, docker/build-push-action@10e90e3645eae34f1e60eeb005ba3a3d33f178e8 #v6, peter-evans/dockerhub-description@1b9a80c056b620d92cedb9d9b5a223409c68ddfa #v5, docker/setup-buildx-action@8d2750c68a42422c14e847fe6c8ac0403b4cbd6f #v3, Noelware/docker-manifest-action@b33ab348026b120a895167160f5605b0197f0862 #0.4.3, softprops/action-gh-release@3bb12739c298aeb8a4eeaf626c5b8d85266b0e65 #v2. (3) Fixed all script injection issues by moving ${{ github.ref_name }}, ${{ matrix.platform }}, ${{ matrix.container.image }}, ${{ matrix.container.tag }}, ${{ steps.platform.outputs.suffix }}, and ${{ env.version }} expressions out of run: shell commands into step-level env: blocks, then referencing them as plain environment variables (GIT_REF_NAME, MATRIX_PLATFORM, CONTAINER_IMAGE, CONTAINER_TAG, PLATFORM_SUFFIX, APP_VERSION) in the shell scripts.

