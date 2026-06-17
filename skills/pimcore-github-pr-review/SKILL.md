---
name: pimcore-github-pr-review
description: Perform a thorough local code review of a checked-out GitHub pull request branch for a Pimcore bundle/repo, using git diff against the base and the local working tree. Use when delegated a PR review (e.g. from the Pimcore Bundle Manager "Review with Claude" button) where the PR branch has already been checked out locally.
metadata:
  audience: pimcore-developers
  focus: github-pr-review
---

## What This Skill Covers

How to produce a thorough, structured **local** code review of a pull request whose
branch has already been checked out into the local working copy:

- Diffing the branch against its base to see exactly what changed.
- Reading the changed files in full context.
- Evaluating correctness, edge cases, error handling, performance, security, and style.
- Running the relevant tests and linters and reporting the results.
- Producing a structured review with severity-tagged, `file:line`-referenced findings.

## When to Use This Skill

Use this when you are asked for a thorough local review of a pull request whose branch is
already checked out — for example when the Pimcore Bundle Manager extension's **Review
with Claude** button is clicked. The delegating prompt supplies the PR number, the
repository (`owner/repo`), the title, the head/base branches, and the local checkout
path; everything else lives here.

To merely assess or work on a PR from its GitHub metadata (without a local checkout), use
the [`pimcore-github-pr-assist`](../pimcore-github-pr-assist/SKILL.md) skill instead.

## 🚫 No GitHub MCP

Use the `gh` CLI and the local working copy for **all** GitHub data. Do **not** use any
GitHub MCP server. Only reach for `gh` when you need PR metadata or existing review
comments:

```bash
gh pr view <number> --repo <repo> --comments
```

## Procedure

Throughout, replace `<number>` with the PR number, `<repo>` with the `owner/repo`, and
`<base>` with the base ref given in the prompt. The base ref is normally
`origin/<baseBranch>`, falling back to `origin/HEAD` when no base branch is given. If the
prompt names a local checkout path, review the working tree there.

### 1. See exactly what changed

```bash
git diff <base>...HEAD
```

Read the changed files in full context — not just the diff hunks.

### 2. Evaluate the change

Evaluate correctness, edge cases, error handling, performance, security, and consistency
with the surrounding code style.

### 3. Run tests and linters

Run the relevant tests and linters if they exist, and report the results.

### 4. Separate blocking from nice-to-have

Separate blocking issues from nice-to-have suggestions.

## Output

Produce a structured review with severity-tagged findings and `file:line` references.
This skill is review-only — do not modify the code; report findings for the author to act
on.
