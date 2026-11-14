---
name: vuln-triage
description:
  Use this skill to triage a reported vulnerability or security issue. It
  reproduces the report, assigns a severity, checks for duplicates and known
  CVEs, and produces a prioritized fix plan with verification steps.
---

# Vulnerability Triage

This skill guides the agent in triaging a reported vulnerability, from first
reproduction through severity rating to a fix plan the maintainer can act on.

## Workflow

### 1. Collect the Report
*   Read the full report: affected component, versions, steps to reproduce,
    and any proof of concept.
*   If the report is a GitHub issue, read the issue and its comments for
    maintainer context and any existing reproduction.
*   If the report is incomplete (no version, no steps, no affected path), list
    exactly what is missing before attempting reproduction.

### 2. Reproduce
*   Set up the reported scenario in a local checkout or sandbox. Do not run a
    proof of concept against a shared or production environment.
*   Confirm whether the vulnerability triggers. Record the exact commands,
    inputs, and observed output.
*   If it does not reproduce, note the versions and environment tried and
    state that clearly instead of guessing at a cause.

### 3. Assess Impact and Severity
*   Determine the impact: what an attacker gains (code execution, data
    access, privilege escalation, denial of service) and who is affected.
*   Rate severity using CVSS where possible, and map it to the project's
    labels: Critical, High, Medium, Low.
*   Distinguish a confirmed vulnerability from a hardening suggestion. A
    hardening suggestion is not a vulnerability and must not be rated as one.

### 4. Check for Duplicates and Known Issues
*   Search open and closed issues and pull requests for the same component
    and the same weakness before filing anything new.
*   Check the dependency's advisory database and CVE records when the report
    involves a third-party package.
*   If a duplicate exists, link to it and stop. Do not open a second report.

### 5. Produce the Fix Plan
*   **Fix**: the smallest change that removes the vulnerability, with the
    file and function to change.
*   **Tests**: a regression test that fails before the fix and passes after
    it. Name the test file to add it to.
*   **Verification**: how to confirm the fix, including `npm run preflight`
    and a re-run of the original proof of concept.
*   **Disclosure**: if the issue is Critical or High, recommend coordinated
    disclosure through the project's security policy instead of a public
    issue, and do not publish exploit details.

## Provide Feedback

### Structure
*   **Summary**: one paragraph — what the vulnerability is, whether it
    reproduced, and its severity.
*   **Reproduction**: the exact steps and observed output.
*   **Severity**: rating with reasoning.
*   **Duplicates**: what was checked and what was found.
*   **Fix plan**: fix, tests, and verification steps in order.

### Tone
*   Be factual and precise. State what was confirmed and what was not.
*   Never inflate severity. A Medium rated as a Critical destroys trust in
    the triage.
