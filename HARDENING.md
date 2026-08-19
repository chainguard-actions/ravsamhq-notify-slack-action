<!-- markdownlint-disable -->

# Hardening Report: ravsamhq--notify-slack-action/2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ravsamhq--notify-slack-action/2.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): GitHub Actions expressions are interpolated directly inside run: shell command strings. In release.yml, `printf "${{ toJson(github) }}"` injects the full GitHub context object into a shell command. In test-and-update.yml, `echo "${{ toJson(github) }}"` does the same, and `git push origin HEAD:${{ github.event.pull_request.head.ref }} --force` injects the attacker-controlled pull request head ref (from a pull_request trigger) directly into a shell command, enabling command injection.

Locations:

- `.github/workflows/release.yml:15`
- `.github/workflows/test-and-update.yml:43`
- `.github/workflows/test-and-update.yml:50`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks. Failing references in release.yml: actions/checkout@v2, actions/setup-node@v3, andymckay/cancel-action@0.3, ncipollo/release-action@v1 (appears twice). Failing references in test-and-update.yml: actions/checkout@v3, actions/setup-node@v3, actions/cache@v2.

Locations:

- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:62`
- `.github/workflows/release.yml:77`
- `.github/workflows/test-and-update.yml:13`
- `.github/workflows/test-and-update.yml:22`
- `.github/workflows/test-and-update.yml:27`

### missing-permissions (severity: medium)

Neither workflow file defines a top-level permissions: key, and no individual job within either file defines a permissions: block. This means the GITHUB_TOKEN is granted its default (broad) permissions for all jobs, violating the principle of least privilege.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/test-and-update.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across release.yml and test-and-update.yml:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks. In release.yml, toJson(github) is now in GITHUB_CONTEXT env var. In test-and-update.yml, toJson(github) is in GITHUB_CONTEXT and github.event.pull_request.head.ref is in PR_HEAD_REF (sanitized with tr -d '\n\r' before use in git push).

2. unpinned-uses: Pinned all 6 action references to full 40-char commit SHAs using lookup_action_sha: actions/checkout@v2→0717577d, actions/checkout@v3→a37ce912, actions/setup-node@v3→3235b876, andymckay/cancel-action@0.3→b9280e3f, ncipollo/release-action@v1→339a8189 (×2), actions/cache@v2→84922603.

3. missing-permissions: Added top-level 'permissions: {}' to both workflows, with job-level overrides granting only what's needed (contents: read for get-admins, contents: write for create-release and test jobs that push commits/tags).

