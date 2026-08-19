<!-- markdownlint-disable -->

# Hardening Report: ravsamhq--notify-slack-action/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ravsamhq--notify-slack-action/v2** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ toJson(github) }}` is interpolated directly inside a `run:` shell command (`printf "${{ toJson(github) }}"`). The entire GitHub context — including attacker-controlled fields like commit messages and branch names — is expanded by the YAML template engine before the shell ever sees it, enabling arbitrary command injection.

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/test-and-update.yml:44`

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.pull_request.head.ref }}` is interpolated directly inside a `run:` shell command (`git push origin HEAD:${{ github.event.pull_request.head.ref }} --force`). The pull request head ref is attacker-controlled and can contain shell metacharacters, enabling command injection.

Locations:

- `.github/workflows/test-and-update.yml:49`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character SHA commits, making the workflow vulnerable to supply-chain attacks if those tags are moved. Failing references in release.yml: `actions/checkout@v2`, `actions/setup-node@v3`, `andymckay/cancel-action@0.3`, `ncipollo/release-action@v1` (×2). Failing references in test-and-update.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `actions/cache@v2`.

Locations:

- `.github/workflows/release.yml:19`
- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:88`
- `.github/workflows/release.yml:97`
- `.github/workflows/test-and-update.yml:10`
- `.github/workflows/test-and-update.yml:20`
- `.github/workflows/test-and-update.yml:27`

### permissions (severity: medium)

missing-permissions: Neither `.github/workflows/release.yml` nor `.github/workflows/test-and-update.yml` declares a top-level `permissions:` key, and no job within either file has a `permissions:` block. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/test-and-update.yml:1`

### github-env-injection (severity: high)

Values derived from repository content and git tags are written to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: `last_version` (from `git describe --tags`), `next_version` (parsed from a file), `major_version` (derived from `next_version`), and `major_version_exists` (from `git rev-parse`) are all written unsanitized. A malicious tag name or file content containing newlines could inject arbitrary environment variables into subsequent steps.

Locations:

- `.github/workflows/release.yml:48`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, permissions, github-env-injection

**Notes:**

Fixed all findings in both workflow files:

1. script-injection: Moved `${{ toJson(github) }}` in release.yml and test-and-update.yml into `env:` blocks as GITHUB_CONTEXT. Moved `${{ github.event.pull_request.head.ref }}` in test-and-update.yml into an `env:` block as PR_HEAD_REF, referenced as "$PR_HEAD_REF" in the git push command.

2. unpinned-uses: Pinned all 7 action references to full 40-character commit SHAs with tag comments: actions/checkout@v2→0717577d, actions/checkout@v3→a37ce912, actions/setup-node@v3→3235b876, andymckay/cancel-action@0.3→b9280e3f, ncipollo/release-action@v1→339a8189 (×2), actions/cache@v2→84922603.

3. missing-permissions: Added `permissions: {}` at the top level of both workflows, with job-level overrides: `contents: read` for get-admins, `contents: write` for create-release and test jobs.

4. github-env-injection: Sanitized all four values written to $GITHUB_ENV using `printf '%s' "$value" | tr -d '\n\r'` before writing: last_version, next_version, major_version, and major_version_exists.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/.github/workflows/release.yml:
1. 'Get Major Version related to Next Version' step (lines 77, 82): Replaced unquoted `echo $next_version` inside backtick substitution with properly quoted `echo "$next_version"` inside `$(...)`, and quoted `$major_version` in `git rev-parse "$major_version"`.
2. 'Update Major Version Release if it exists' step (lines 107-108): Quoted `$major_version` in both `git tag -fa "$major_version"` and `git push origin "$major_version" --force` to prevent shell metacharacter injection from attacker-controlled git tag/commit message values.

