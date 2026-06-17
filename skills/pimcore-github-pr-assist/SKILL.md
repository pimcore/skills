---
name: pimcore-github-pr-assist
description: Assess and work on a GitHub pull request for a Pimcore bundle/repo using the gh CLI and the local working copy. Use when delegated a PR (e.g. from the Pimcore Bundle Manager "Delegate to Claude" button on a pull request) to understand it, judge correctness/design, and propose next steps. For a deep local diff review use pimcore-github-pr-review instead.
metadata:
  audience: pimcore-developers
  focus: github-pr-assist
---

## What This Skill Covers

How to help someone work on an open pull request:

- Gathering the PR's context (description, comments, diff, CI checks) from the `gh` CLI.
- Summarizing what the PR changes and the intent behind it.
- Assessing correctness, design, and whether it fully addresses the linked issue.
- Identifying bugs, missing edge cases, absent tests, and security concerns.
- Proposing concrete improvements or next steps.

## When to Use This Skill

Use this when you are asked to work on or assess a specific pull request — for example
when the Pimcore Bundle Manager extension's **Delegate to Claude** button is clicked on a
PR. The delegating prompt supplies the PR number, the repository (`owner/repo`), the
title, and the head/base branches; everything else lives here.

For a thorough line-by-line **local** code review of a checked-out branch, use the
[`pimcore-github-pr-review`](../pimcore-github-pr-review/SKILL.md) skill instead.

## 🚫 No GitHub MCP

Use the `gh` CLI and the local working copy for **all** GitHub data. Do **not** use any
GitHub MCP server. If a `gh` command fails (auth, network), report the failure plainly
rather than substituting another data source.

## Procedure

Throughout, replace `<number>` with the PR number and `<repo>` with the `owner/repo`
given to you in the prompt.

### 1. Gather context

```bash
gh pr view <number> --repo <repo> --comments
gh pr diff <number> --repo <repo>
gh pr checks <number> --repo <repo>
```

### 2. Summarize the change

Summarize what the PR changes and the intent behind it.

### 3. Assess correctness and design

Assess correctness and design, and whether the PR fully addresses the linked issue.

### 4. Identify problems

Identify bugs, missing edge cases, absent tests, and security concerns, each with a
`file:line` reference.

### 5. Propose next steps

Propose concrete improvements or next steps.

## Stop Before Changing Anything

Work in **plan mode** and wait for explicit approval before editing any files. Your first
response is analysis and a proposed plan, not an implementation.
