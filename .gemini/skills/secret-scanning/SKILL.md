---
name: secret-scanning
description:
  Use this skill to scan code and repository history for leaked secrets such as
  API keys, tokens, passwords, and private keys. It covers staged and working
  tree changes, committed history, and configuration files, and reports each
  finding with its location, the type of secret, and the remediation step.
---

# Secret Scanning

This skill guides the agent in finding secrets that were accidentally committed
or left in the working tree, and in removing them safely without breaking the
build.

## Workflow

### 1. Determine Scan Target
*   **Local Changes**: If the user asks to "scan my changes", target staged and
    unstaged changes in the working tree.
*   **History**: If the user asks to scan history, a branch, or the whole
    repository, target the commit range they name. Default to the current
    branch against `main` when no range is given.
*   **Path**: If the user names a directory or file, scan only that path.

### 2. Identify Changes
*   Check status: `git status`
*   Read diffs: `git diff` (working tree) and/or `git diff --staged` (staged).
*   For history: `git log --all -p -S <pattern>` for a specific pattern, or
    `git log --oneline <base>..HEAD` to list the commits in scope.

### 3. Scan for Secret Patterns
Search the target for the following, using `rg` (ripgrep) or `git grep` so
binary and vendored files are skipped:

*   **API keys and tokens**: `AKIA[0-9A-Z]{16}`, `ghp_`, `gho_`, `github_pat_`,
    `sk-`, `AIza`, `xox[baprs]-`, `-----BEGIN .* PRIVATE KEY-----`.
*   **Credentials in code**: `password =`, `passwd`, `api_key`, `secret`,
    `token`, `client_secret`, `aws_secret_access_key` assigned to a literal
    value rather than read from the environment.
*   **Configuration files**: `.env`, `*.pem`, `*.key`, `credentials.json`,
    `service-account.json`, `.npmrc` with an auth token, and any file that
    contains a real credential instead of a placeholder.
*   **URLs with embedded credentials**: `https://user:password@host`.
*   **High-entropy strings**: long base64 or hex literals that do not match a
    known hash, test fixture, or example value.

Treat the following as **not** findings: placeholders (`<redacted>`,
`YOUR_API_KEY`, `example`, `test`), values in test fixtures that are clearly
fake, and public keys. When in doubt, flag it and say why it looks suspicious
rather than silently skipping it.

### 4. Verify Each Finding
*   Confirm the match is a real secret, not a placeholder or a hash of public
    data. Check whether the value appears in committed history or only in the
    working tree.
*   Check whether the file is covered by `.gitignore`. A secret in an ignored
    file is still a finding if the file is tracked or was ever committed.
*   Do **not** print the full secret value in output. Show only the file,
    line, secret type, and a redacted prefix (first 4 characters at most).

### 5. Remediate
*   **Working tree only**: remove the secret from the file and read it from
    the environment instead (for example, `process.env.GEMINI_API_KEY`). Then
    run `npm run preflight` to confirm nothing broke.
*   **Already committed**: tell the user the secret must be revoked or rotated
    at the provider first, because removing it from git history does not
    un-expose it. Suggest `git filter-repo` or BFG to purge it from history,
    and warn that rewriting shared history requires coordinating with everyone
    who has pulled the branch.
*   **Prevention**: suggest adding the file pattern to `.gitignore` and, where
    the project supports it, a pre-commit hook that blocks new secrets.

## Provide Feedback

### Structure
*   **Summary**: how many findings, and whether any are in committed history.
*   **Findings**:
    *   **Critical**: a real, live-looking secret in committed history.
    *   **Warning**: a real secret in the working tree or staged changes.
    *   **Info**: suspicious patterns that turned out to be placeholders.
*   **Remediation**: the exact next step for each finding.

### Tone
*   Be direct and factual. Never reproduce a full secret value in any output,
    comment, or commit message.
