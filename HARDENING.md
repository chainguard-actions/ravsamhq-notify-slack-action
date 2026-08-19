<!-- markdownlint-disable -->

# Hardening Report: ravsamhq--notify-slack-action/2.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **ravsamhq--notify-slack-action/2.2.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct ${{ }} expression interpolation inside run: shell blocks. In release.yml, `printf "${{ toJson(github) }}"` dumps the entire GitHub context into a shell command (sub-rule a). In test-and-update.yml, `echo "${{ toJson(github) }}"` similarly interpolates the full context, and `git push origin HEAD:${{ github.event.pull_request.head.ref }}` injects the attacker-controlled PR branch name directly into a shell command — this is triggered on pull_request events, making it exploitable by a PR author (sub-rule a).

Locations:

- `.github/workflows/release.yml:14`
- `.github/workflows/test-and-update.yml:40`
- `.github/workflows/test-and-update.yml:46`

### unpinned-uses (severity: high)

Multiple `uses:` references in workflow files use mutable tag or version refs instead of pinned 40-character SHA digests. In release.yml: `actions/checkout@v2`, `actions/setup-node@v3`, `andymckay/cancel-action@0.2`, `ncipollo/release-action@v1` (×2). In test-and-update.yml: `actions/checkout@v3`, `actions/setup-node@v3`, `actions/cache@v2`. Any of these tags could be moved to point to malicious code.

Locations:

- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:32`
- `.github/workflows/release.yml:37`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:84`
- `.github/workflows/release.yml:93`
- `.github/workflows/test-and-update.yml:12`
- `.github/workflows/test-and-update.yml:16`
- `.github/workflows/test-and-update.yml:21`

### missing-permissions (severity: medium)

Neither workflow file declares a top-level `permissions:` key, and no job within either file declares job-level `permissions:`. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often `write-all` for older repositories), granting excessive access. Both release.yml and test-and-update.yml are affected.

Locations:

- `.github/workflows/release.yml:1`
- `.github/workflows/test-and-update.yml:1`

### github-env-injection (severity: high)

In release.yml, several run: steps write values to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). The values written include: `last_version` (from `git describe --tags`), `next_version` (parsed from semantic-release output), `major_version` and `major_version_exists` (derived from the above). These values are sourced from git tags and file content that could be attacker-influenced (e.g. via a crafted tag name containing newlines), allowing injection of arbitrary environment variables into subsequent steps via GITHUB_ENV protocol injection.

Locations:

- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:71`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions, github-env-injection

**Notes:**

Fixed all four findings across both workflow files:

1. script-injection: Moved all ${{ }} expressions out of run: shell blocks into step env: blocks. In release.yml, toJson(github) is now in GITHUB_CONTEXT env var. In test-and-update.yml, toJson(github) is in GITHUB_CONTEXT and github.event.pull_request.head.ref is in PR_HEAD_REF, referenced as "$PR_HEAD_REF" in the git push command.

2. unpinned-uses: Pinned all 6 action references to full 40-char SHAs (resolved via lookup_action_sha): actions/checkout@v2→ee0669bd, actions/checkout@v3→f43a0e5f, actions/setup-node@v3→3235b876, andymckay/cancel-action@0.2→8f8510d9, ncipollo/release-action@v1→339a8189 (×2), actions/cache@v2→84922603.

3. missing-permissions: Added top-level 'permissions: contents: write' to both workflow files. contents:write is the minimum needed for creating releases (release.yml) and pushing commits (test-and-update.yml).

4. github-env-injection: All four GITHUB_ENV writes in release.yml now sanitize values through 'printf \'%s\' "$val" | tr -d \'\n\r\'' before writing, preventing newline-based protocol injection attacks.

