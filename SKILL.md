---
name: git-split-publish
description: Inspect a mixed git worktree, split pending changes into task-based commits, push each commit in order, and open a draft PR only when explicitly requested. Acts on what the request already decides, asks once with concrete options about what it leaves genuinely open, and leads every report with the outcome.
version: 0.1.1
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

Use this skill when worktree changes are mixed and the user wants commits separated by task before publishing.

Operating stance: the publish request itself authorizes staging, committing, and pushing the changes it covers. Act on what the request already decides; reserve questions for decisions the request leaves genuinely open, and ask them once, batched, with concrete options. Finish the whole loop rather than stopping after the first group, and lead the final report with the outcome.

## Workflow

Follow `references/workflow.md` for the full procedure and `references/grouping-rules.md` for the commit-boundary rubric. The core loop:

1. **Inspect state first** — evidence before any state change: `git status -sb`, `git diff --stat`, `git branch --show-current`, `git remote -v`. Before staging or committing for a publish, `git remote get-url origin` and `git ls-remote origin` must succeed, else stop and report the blocker with the concrete next action. Clean worktree → report nothing to publish and stop.
2. **Read local constraints** — root `AGENTS.md` if present, plus branch/release/validation docs relevant to the changed files.
3. **Scope from the request** — a whole-worktree request sets the scope; a request naming specific work leaves unrelated changes untouched (say what was left out). Never `git add -A`; stage explicit paths only.
4. **Group before staging** — one coherent task per commit; order groups so every commit stands on its own; when a file's hunks span groups, split with `git add -p`/`git add -e` or commit under the dominant intent and disclose the rider. Decide clearly defensible groupings and say so; batch the genuinely ambiguous files into one question with concrete options — never group-by-group, never "shall I proceed?".
5. **Commit and push one group at a time** — verify staged scope (`git diff --cached --stat`), validate and gate the push on the result, push in order, retry transient network failures up to 4 times with exponential backoff (auth failures are blockers, not retries), and finish every group or report exactly which were pushed and which were not.
6. **PRs only on request** — no auto-PRs; open a draft PR only when the user asks and the branch is not the default. Prefer repo-native tooling; fallback `gh pr create --draft`.

## Guardrails

These hold regardless of how autonomous the run is:

- Do not silently include unrelated files.
- Do not rewrite or discard user changes.
- Do not amend/squash/reorder existing commits unless requested.
- Do not stage broadly (`git add -A`, `git add .`).
- Screen every path before staging. Never stage secrets or local env files
  (`.env`, `.env.*`, `*.pem`, `*.key`, credentials, tokens), dependency, build,
  or cache artifacts (`node_modules/`, `__pycache__/`, `dist/`, `build/`,
  `.venv/`, `*.log`), or large or machine-generated blobs (roughly >5 MB, or
  anything vendored or generated). Exclude such a file from the commit and flag
  it in the report; if a secret appears to be committed already, stop and warn
  instead of pushing. Respect `.gitignore` — do not force-add ignored paths.
- Stop before staging or committing when remote/auth is missing or
  inaccessible, and report the concrete next action.

## Reporting

Lead with the outcome: the first sentence answers "what happened" — for
example "Pushed 3 commits to origin/feature-x" or "Stopped before committing:
origin rejects authentication". Then cover, in readable prose rather than
fragment checklists:

- current branch and upstream state
- each commit group executed or proposed: hash, message, files, push result
- decisions made without asking (such as where an ambiguous file went), so the
  user can veto them after the fact
- what was intentionally left unstaged and why
- validation run and its result
- PR skipped/created decision and reason
- remaining blockers, reported faithfully: include the actual error output,
  and never claim success for anything unverified

## Examples

- "commit and push but split by job"
- "split these changes into separate commits and publish"
- "group pending changes by task then push"
