---
name: git-split-publish
description: Safely inspect a mixed git worktree, split pending changes into task-based commits, push each commit in order, and optionally open a draft PR when explicitly requested.
version: 0.1.0
author: suhohan
homepage: https://github.com/suho-han/git-split-publish
tags:
  - git
  - workflow
  - productivity
  - automation
  - devtools
---

# Split Publish

## Overview

Use this skill when worktree changes are mixed and the user wants commits separated by job before publishing.

## Workflow

1. Inspect repository state first.

- Run `git status -sb`, `git diff --stat`, `git branch --show-current`, and `git remote -v`.
- If push may be required, run `git status --porcelain=v2 --branch`.
- Before staging or committing for a publish request, verify that a push target
  exists: `git remote get-url origin` and `git ls-remote origin` must succeed.
  If the remote is missing, inaccessible, or auth is unavailable, stop before
  staging or committing and report the concrete blocker.
- If worktree is clean, stop and report there is nothing to publish.

2. Load repository guidance before grouping.

- Read root `AGENTS.md` if present.
- Read local docs for branch/release/validation policy when relevant.

3. Propose commit groups.

- Inspect diffs per file with `git diff -- <path>` when boundaries are unclear.
- Group by one coherent task per commit.
- For each group, provide:
  - label
  - exact file list
  - commit message
  - push/PR intent
- If grouping is ambiguous, ask the user.

4. Confirm scope before staging.

- On mixed worktree, get explicit confirmation before staging.
- Do not default to `git add -A`.
- Use explicit path staging: `git add -- <path>...`.

5. Commit and push group-by-group.

- Stage only approved files.
- Verify staged scope using `git diff --cached --stat` (and `git diff --cached` if needed).
- Commit with terse task-level message.
- Run minimal relevant validation before first push.
- Push each commit in order:
  - if no upstream: `git push -u origin $(git branch --show-current)`
  - else: `git push`

6. Open draft PR only when explicitly requested.

- Do not open PR automatically.
- Open only when both are true:
  - branch is not default branch
  - user explicitly asks for PR (or major-change PR)
- Prefer repo-native tooling; fallback to `gh pr create --draft`.

## Guardrails

- Do not silently include unrelated files.
- Do not rewrite/discard user changes.
- Do not amend/squash/reorder existing commits unless requested.
- Stop before staging or committing when remote/auth is missing or inaccessible.

## Output

Report:

- current branch and upstream state
- proposed or executed commit groups
- created commit hashes/messages
- push result per commit
- PR skipped/created decision and reason
- validation run
- remaining blockers or assumptions

## Examples

- "commit and push but split by job"
- "split these changes into separate commits and publish"
- "group pending changes by task then push"
