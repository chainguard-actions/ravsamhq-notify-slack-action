<!-- markdownlint-disable -->

# Hardening Report: ravsamhq--notify-slack-action/2.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ravsamhq--notify-slack-action/2.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings, allowing template substitution before the shell ever sees the value.

1. release.yml line 14: `printf "${{ toJson(github) }}"` — the entire github context object is injected directly into a shell printf command.

2. test-and-update.yml line 48: `echo "${{ toJson(github) }}"` — the entire github context object is injected directly into a shell echo command.

3. test-and-update.yml line 54: `git push origin HEAD:${{ github.event.pull_request.head.ref }} --force` — the attacker-controlled PR head branch name is interpolated directly into a git push command. A branch name containing shell metacharacters or newlines could alter the command.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/test-and-update.yml:48`
- `.github/workflows/test-and-update.yml:54`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tag is moved or the repository is compromised.

In .github/workflows/release.yml:
- `uses: actions/checkout@v2` (line 21)
- `uses: actions/checkout@v2` (line 33)
- `uses: actions/setup-node@v3` (line 40)
- `uses: andymckay/cancel-action@0.3` (line 56)
- `uses: ncipollo/release-action@v1` (line 78)
- `uses: ncipollo/release-action@v1` (line 88)

In .github/workflows/test-and-update.yml:
- `uses: actions/checkout@v3` (line 12)
- `uses: actions/setup-node@v3` (line 22)
- `uses: actions/cache@v2` (line 27)

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:56`
- `.github/workflows/release.yml:78`
- `.github/workflows/release.yml:88`
- `.github/workflows/test-and-update.yml:12`
- `.github/workflows/test-and-update.yml:22`
- `.github/workflows/test-and-update.yml:27`

### missing-permissions (severity: medium)

Neither workflow file has a top-level `permissions:` key, and no job within either file defines its own `permissions:` block. Without explicit permissions, workflows run with the default repository permissions (which may include write access to contents, pull-requests, etc.), violating the principle of least privilege.

- .github/workflows/release.yml: no top-level or job-level permissions defined across jobs `get-admins` and `create-release`.
- .github/workflows/test-and-update.yml: no top-level or job-level permissions defined for job `test`.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/test-and-update.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across both workflow files:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks. In release.yml, toJson(github) is now in GITHUB_CONTEXT env var. In test-and-update.yml, toJson(github) is in GITHUB_CONTEXT and github.event.pull_request.head.ref is in PR_HEAD_REF, both referenced as plain shell variables.

2. unpinned-uses: Pinned all 9 action references to full 40-character commit SHAs with tag comments: actions/checkout@v2→0717577d, actions/checkout@v3→a37ce912, actions/setup-node@v3→3235b876, andymckay/cancel-action@0.3→b9280e3f, ncipollo/release-action@v1→339a8189 (×2), actions/cache@v2→84922603.

3. missing-permissions: Added top-level `permissions: {}` to both files, plus job-level permissions: get-admins gets contents:read, create-release gets contents:write (for release creation and tag pushing), test job gets contents:write (for committing dist changes).

