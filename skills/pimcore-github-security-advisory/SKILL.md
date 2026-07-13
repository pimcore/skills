---
name: pimcore-github-security-advisory
description: Assess and remediate a GitHub security advisory (GHSA/CVE) for a Pimcore bundle/repo using the gh CLI and the local working copy. Use when delegated a security advisory (e.g. from the Pimcore Bundle Manager "Delegate to Claude" button on an advisory) to explain the vulnerability, check if the workspace is affected, and recommend a remediation.
metadata:
  audience: pimcore-developers
  focus: github-security-advisory
---

## What This Skill Covers

How to assess and remediate a single GitHub security advisory:

- Fetching the full advisory from the `gh` CLI.
- Explaining the vulnerability and its real-world impact for this codebase.
- Checking whether the current workspace is affected (installed versions of the affected
  packages).
- Recommending a concrete remediation: safe target versions, plus config or code
  mitigations, with step-by-step instructions and the breaking changes to watch for.

## When to Use This Skill

Use this when you are asked to assess or remediate a specific security advisory — for
example when the Pimcore Bundle Manager extension's **Delegate to Claude** button is
clicked on an advisory. The delegating prompt supplies the advisory id (`GHSA-…`), the CVE
id when assigned, the repository (`owner/repo`), the summary, and the affected
package(s); everything else lives here.

## 🚫 No GitHub MCP

Use the `gh` CLI and the local working copy for **all** GitHub data. Do **not** use any
GitHub MCP server. If a `gh` command fails (auth, network), report the failure plainly
rather than substituting another data source.

## Procedure

Throughout, replace `<id>` with the advisory id (`GHSA-…`) and `<repo>` with the
`owner/repo` given to you in the prompt.

### 1. Fetch the full advisory

```bash
gh api repos/<repo>/security-advisories/<id>
```

### 2. Explain the vulnerability

Explain the vulnerability and its real-world impact **for this codebase** — not just the
generic description.

### 3. Check whether this workspace is affected

Inspect the installed versions of the affected package(s) in `composer.lock` and the
`vendor/` directory, and state clearly whether this workspace is affected.

### 4. Recommend a remediation

Recommend a concrete remediation: the safe target version(s) to upgrade to, plus any
config or code mitigations, with step-by-step instructions.

### 5. Call out breaking changes

Call out any breaking changes the upgrade may introduce and the tests to run afterwards.

## Stop Before Changing Anything

Work in **plan mode** and wait for explicit approval before changing anything. Your first
response is analysis and a proposed remediation plan, not an implementation.
