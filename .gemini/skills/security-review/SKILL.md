---
name: security-review
description:
  Use this skill to review code for security vulnerabilities. It supports both
  local changes (staged or working tree) and remote Pull Requests (by ID or
  URL). It focuses on injection, authentication and authorization, data
  exposure, and unsafe use of shell, network, and filesystem APIs.
---

# Security Reviewer

This skill guides the agent in conducting a focused security review of code
changes for both local development and remote Pull Requests. It complements
the general code review skill: where that skill covers correctness and
maintainability, this one covers only whether the change can be abused.

## Workflow

### 1. Determine Review Target
*   **Remote PR**: If the user provides a PR number or URL (e.g., "Security
    review PR #123"), target that remote PR.
*   **Local Changes**: If no specific PR is mentioned, or if the user asks to
    "security review my changes", target the current local file system states
    (staged and unstaged changes).

### 2. Preparation

#### For Remote PRs:
1.  **Checkout**: Use the GitHub CLI to checkout the PR.
    ```bash
    gh pr checkout <PR_NUMBER>
    ```
2.  **Preflight**: Execute the project's standard verification suite to catch automated failures early.
    ```bash
    npm run preflight
    ```
3.  **Context**: Read the PR description and any existing comments to understand the goal and history.

#### For Local Changes:
1.  **Identify Changes**:
    *   Check status: `git status`
    *   Read diffs: `git diff` (working tree) and/or `git diff --staged` (staged).
2.  **Preflight (Optional)**: If the changes are substantial, ask the user if they want to run `npm run preflight` before reviewing.

### 3. In-Depth Analysis
Analyze the code changes against the following checks. Only report issues
with a concrete attack path in this codebase; do not report theoretical issues
that cannot be reached.

*   **Command injection**: user input, file contents, or model output passed
    to a shell, `child_process.exec`, or a command substitution without
    quoting, allow-listing, or an argument array. Check both direct calls and
    helpers that build command strings.
*   **Path traversal**: user-controlled or model-controlled paths used with
    `fs` APIs without resolving them against an allowed root and rejecting
    `..` escapes and symlinks that point outside it.
*   **Injection into prompts and tools**: untrusted content (tool output,
    file contents, web pages) concatenated into prompts or tool arguments in a
    way that lets it override instructions or exfiltrate data.
*   **Authentication and authorization**: new endpoints, tools, or commands
    that skip the existing auth check; credentials compared with `==` instead
    of a timing-safe comparison; tokens or session data stored in plaintext
    or logged.
*   **Data exposure**: secrets, tokens, or personal data written to logs,
    error messages, telemetry, or user-facing output. See the secret-scanning
    skill for leaked-credential checks.
*   **Network and SSRF**: URLs built from user or model input fetched without
    an allow-list or without blocking private and link-local addresses.
*   **Deserialization and parsing**: `eval`, `Function`, `JSON.parse` on
    untrusted input without schema validation, or XML/YAML parsers that
    resolve external entities.
*   **Dependencies**: a new dependency with a known vulnerability, or a
    dependency pinned to a version range that can float to an unreviewed
    release. Check `package-lock.json` for the resolved version.

### 4. Provide Feedback

#### Structure
*   **Summary**: a high-level overview of the security posture of the change.
*   **Findings**:
    *   **Critical**: an exploitable vulnerability with a concrete attack path.
    *   **High**: a likely vulnerability that needs a fix before merge.
    *   **Medium**: a hardening gap that reduces defense in depth.
    *   **Nitpicks**: minor issues (optional).
*   **Conclusion**: Clear recommendation (Approved / Request Changes).

#### Tone
*   Be constructive, professional, and friendly.
*   For every finding, explain the attack path, the impact, and a concrete
    fix. Do not report a finding without a fix.
*   For approvals, acknowledge the specific value of the contribution.

### 5. Cleanup (Remote PRs only)
*   After the review, ask the user if they want to switch back to the default branch (e.g., `main` or `master`).
