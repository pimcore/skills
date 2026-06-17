---
name: pimcore-github-issue
description: Investigate and resolve a GitHub issue for a Pimcore bundle/repo using the gh CLI and the local working copy. Use when delegated a GitHub issue (e.g. from the Pimcore Bundle Manager "Delegate to Claude" button on an issue) to triage, find the root cause, and propose a fix.
metadata:
  audience: pimcore-developers
  focus: github-issue-triage
---

## What This Skill Covers

How to take a single GitHub issue from "reported" to "understood, with a concrete fix proposed":

- Gathering the full issue context (body, comments, linked PRs) from the `gh` CLI.
- Reproducing the problem against the code in the current workspace.
- Locating the relevant code and explaining the most likely root cause.
- Proposing a concrete fix (or a few options with trade-offs) and the tests to cover it.

## When to Use This Skill

Use this when you are asked to investigate or resolve a specific GitHub issue — for
example when the Pimcore Bundle Manager extension's **Delegate to Claude** button is
clicked on an issue. The delegating prompt supplies the issue number, the repository
(`owner/repo`), and the title; everything else lives here.

## 🚫 No GitHub MCP

Use the `gh` CLI and the local working copy for **all** GitHub data. Do **not** use any
GitHub MCP server. If a `gh` command fails (auth, network), report the failure plainly
rather than substituting another data source.

## Procedure

Throughout, replace `<number>` with the issue number and `<repo>` with the `owner/repo`
given to you in the prompt.

### 1. Gather context

```bash
gh issue view <number> --repo <repo> --comments
```

Then inspect any linked pull requests and the related code in this workspace. Read the
referenced files in full context — not just the snippets quoted in the issue.

### 2. Summarize the problem

State the problem in your own words: the expected behavior versus the actual behavior,
the affected versions/components, and the reproduction steps if any are given.

### 3. Find the root cause

Locate the relevant code in this workspace and explain the **most likely root cause**.
Trace the actual code path rather than guessing from the symptom. Cite `file:line`
references for every claim.

### 4. Propose a fix

Propose a concrete fix — or a few options with their trade-offs — and outline the
implementation steps. Identify the tests that should cover the change (existing tests to
extend, or new ones to add).

## Stop Before Changing Anything

Work in **plan mode** and wait for explicit approval before editing any files. Your first
response is analysis and a proposed plan, not an implementation.
