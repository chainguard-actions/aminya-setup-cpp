<!-- markdownlint-disable -->

# Hardening Report: aminya--setup-cpp/v1.8.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aminya--setup-cpp/v1.8.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in CI.yml directly interpolate ${{ ... }} expressions into shell commands, violating rule (a). This allows expression values to be parsed as shell metacharacters before the shell ever sees them.

1. 'Update Dist' step (line 57): `if [[ "${{ github.ref_name }}" =~ ^refs/heads/ ]]` — github.ref_name is attacker-controllable via branch names.

2. 'Define Platform Suffix' step (line 291): `if [[ "${{ matrix.platform }}" == "linux/amd64" ]]` — matrix.platform is a workflow-controllable context.

3. 'Tag latest locally' steps (lines ~321-322, ~338-339, ~360-361, ~383-384): `docker tag aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}-${{ env.version }}-${{ steps.platform.outputs.suffix }} ...` — multiple matrix and steps contexts interpolated directly.

4. 'Push latest to Docker Hub' steps (lines ~327-328, ~345-346, ~367-368, ~390-391): `docker push aminya/${{ matrix.container.image }}:${{ matrix.container.tag }}` — matrix contexts interpolated directly.

Locations:

- `.github/workflows/CI.yml:57`
- `.github/workflows/CI.yml:291`
- `.github/workflows/CI.yml:321`
- `.github/workflows/CI.yml:327`
- `.github/workflows/CI.yml:338`
- `.github/workflows/CI.yml:345`
- `.github/workflows/CI.yml:360`
- `.github/workflows/CI.yml:367`
- `.github/workflows/CI.yml:383`
- `.github/workflows/CI.yml:390`

### github-env-injection (severity: high)

The 'Define Platform Suffix' step writes to $GITHUB_OUTPUT using a value derived from ${{ matrix.platform }} that is directly interpolated into the run: block without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write). An attacker who can control matrix values could inject newlines into $GITHUB_OUTPUT to poison subsequent step outputs.

Offending lines:
  if [[ "${{ matrix.platform }}" == "linux/amd64" ]]; then
    echo "suffix=amd64" >> $GITHUB_OUTPUT
  else
    echo "suffix=arm64" >> $GITHUB_OUTPUT
  fi

Locations:

- `.github/workflows/CI.yml:291`
- `.github/workflows/CI.yml:292`
- `.github/workflows/CI.yml:294`

### unpinned-uses (severity: high)

All uses: references in CI.yml use mutable version tags or version strings instead of immutable 40-character SHA commit hashes. This exposes the workflow to supply-chain attacks if any referenced action is compromised or its tag is moved.

Failing references:
- actions/checkout@v6 (lines 27, 96, 175, 279)
- actions/setup-node@v6 (lines 32, 104, 185, 220, 239)
- oven-sh/setup-bun@v2 (lines 37, 109, 190)
- actions/upload-artifact@v6 (lines 72, 148)
- actions/download-artifact@v7 (lines 99, 180, 284)
- actions/cache@v5 (lines 115, 128, 196)
- rharkor/caching-for-turbo@v1.8 (line 125)
- docker/login-action@v3 (lines 298, ~420)
- docker/build-push-action@v6 (multiple lines in Docker job)
- peter-evans/dockerhub-description@v5 (multiple lines in Docker job)
- docker/setup-buildx-action@v3 (Docker-Manifest job)
- Noelware/docker-manifest-action@0.4.3 (Docker-Manifest job, 3 occurrences)
- softprops/action-gh-release@v2 (Release job)

Locations:

- `.github/workflows/CI.yml:27`
- `.github/workflows/CI.yml:32`
- `.github/workflows/CI.yml:37`
- `.github/workflows/CI.yml:72`
- `.github/workflows/CI.yml:96`
- `.github/workflows/CI.yml:99`
- `.github/workflows/CI.yml:104`
- `.github/workflows/CI.yml:109`
- `.github/workflows/CI.yml:115`
- `.github/workflows/CI.yml:125`
- `.github/workflows/CI.yml:128`
- `.github/workflows/CI.yml:148`
- `.github/workflows/CI.yml:180`
- `.github/workflows/CI.yml:185`
- `.github/workflows/CI.yml:190`
- `.github/workflows/CI.yml:196`
- `.github/workflows/CI.yml:220`
- `.github/workflows/CI.yml:239`
- `.github/workflows/CI.yml:279`
- `.github/workflows/CI.yml:284`
- `.github/workflows/CI.yml:298`
- `.github/workflows/CI.yml:306`

### missing-permissions (severity: medium)

The workflow file CI.yml has no top-level permissions: key and none of its jobs (Build, BuildExecutable, Test, Docker, Docker-Manifest, Release) define a job-level permissions: block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be broader than necessary (e.g., write access to contents, packages, etc.).

Locations:

- `.github/workflows/CI.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four security findings in .github/workflows/CI.yml:

1. unpinned-uses: Pinned all 13 action references to full SHA hashes with tag comments: actions/checkout@d23441a48e516b6c34aea4fa41551a30e30af803 (v6), actions/setup-node@249970729cb0ef3589644e2896645e5dc5ba9c38 (v6), oven-sh/setup-bun@0c5077e51419868618aeaa5fe8019c62421857d6 (v2), actions/upload-artifact@b7c566a772e6b6bfb58ed0dc250532a479d7789f (v6), actions/download-artifact@37930b1c2abaa49bbe596cd826c3c89aef350131 (v7), actions/cache@caa296126883cff596d87d8935842f9db880ef25 (v5), rharkor/caching-for-turbo@a1c4079258ae08389be75b57d4d7a70f23c1c66d (v1.8), docker/login-action@c94ce9fb468520275223c153574b00df6fe4bcc9 (v3), docker/build-push-action@10e90e3645eae34f1e60eeb005ba3a3d33f178e8 (v6), peter-evans/dockerhub-description@1b9a80c056b620d92cedb9d9b5a223409c68ddfa (v5), docker/setup-buildx-action@8d2750c68a42422c14e847fe6c8ac0403b4cbd6f (v3), Noelware/docker-manifest-action@b33ab348026b120a895167160f5605b0197f0862 (0.4.3), softprops/action-gh-release@3bb12739c298aeb8a4eeaf626c5b8d85266b0e65 (v2).

2. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. github.ref_name → REF_NAME; matrix.platform → MATRIX_PLATFORM; matrix.container.image/tag and steps.platform.outputs.suffix → CONTAINER_IMAGE/CONTAINER_TAG/PLATFORM_SUFFIX/APP_VERSION in all docker tag/push steps.

3. github-env-injection: The 'Define Platform Suffix' step now compares $MATRIX_PLATFORM (from env:) and writes a hardcoded safe literal ('amd64' or 'arm64') to $GITHUB_OUTPUT using printf, eliminating any possibility of newline injection.

4. missing-permissions: Added top-level 'permissions: {}' (deny-all) and job-level permissions: Build/Release get 'contents: write'; BuildExecutable/Test/Docker/Docker-Manifest get 'contents: read'.

