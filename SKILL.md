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

1. Inspect repository state first — evidence before any state change.

- Run `git status -sb`, `git diff --stat`, `git branch --show-current`, and `git remote -v`.
- If push may be required, run `git status --porcelain=v2 --branch`.
- Before staging or committing for a publish request, verify that a push target
  works: `git remote get-url origin` and `git ls-remote origin` must succeed.
  If the remote is missing, unreachable, or auth is unavailable, stop before
  staging or committing and report the blocker with the concrete next action
  (for example "run `gh auth login`" or "add a remote with
  `git remote add origin <url>`"). Never leave a local commit behind for a
  publish request that was already known to be unpushable.
- If the worktree is clean, report that there is nothing to publish and stop.

2. Load repository guidance before grouping.

- Read root `AGENTS.md` if present.
- Read local docs for branch/release/validation policy when relevant.

3. Determine scope from the request — do not re-ask what the user already decided.

- If the request covers the whole worktree ("split all my changes and push"),
  the scope is the whole worktree; proceed without asking for confirmation.
- If the request names specific work and unrelated changes also exist, publish
  only what the request covers, leave the rest untouched, and say in the final
  report exactly what was left out.
- Never default to `git add -A`. Use explicit path staging: `git add -- <path>...`.

4. Propose commit groups — decide what you can, batch what you cannot.

- Group by one coherent task per commit, following `references/grouping-rules.md`.
- Inspect per-file diffs with `git diff -- <path>` when a boundary is unclear;
  read the diff before asking about it. `git diff -- <path>` shows nothing for
  untracked files — inspect those with `git diff --no-index -- /dev/null <path>`
  or by reading the file directly.
- Order groups so every commit builds and passes on its own: when several
  changes share a new symbol or helper, give that shared code its own
  foundational commit first (or fold it into the earliest change that needs it)
  so no commit references something a later commit introduces.
- When one file's hunks belong to different groups, split them with `git add -p`
  (or `git add -e` for a contiguous hunk that patch mode refuses to split). If
  they genuinely cannot be separated, commit the file once under its dominant
  intent and disclose the rider hunk in the report.
- For each group, record: label, exact file list, commit message, push/PR intent.
- When one grouping is clearly defensible, choose it and state the choice in
  the final report so the user can veto it after the fact.
- Collect the genuinely ambiguous assignments — files that plausibly belong to
  two groups where the choice changes what gets published — and ask about all
  of them in a single question with concrete options. Never confirm
  group-by-group, and never ask an open-ended "shall I proceed?".

5. Commit and push group-by-group — finish the whole loop.

- Stage each group's exact paths only.
- Verify staged scope using `git diff --cached --stat` (and `git diff --cached`
  if the stat is not enough) before committing.
- Commit with a terse, task-level message.
- Validate before pushing and gate the push on the result: if the change breaks
  validation, stop before pushing and report the actual output — never push or
  claim success on failed or unverified work.
- A single validation before the first push only smoke-tests the combined
  worktree; later groups are still present as uncommitted work, so it does not
  prove any one commit builds on its own. When history must stay bisectable and
  commits are interdependent, validate the at-risk commit in isolation before
  pushing it — build its committed tree (`git archive <commit>` into a temp dir)
  or park the later groups (`git stash -u`) so the check sees only that commit's
  content.
- Distinguish validation that cannot run (missing deps, broken harness or config
  — report as a caveat and fall back to the cheapest sound check such as
  build/typecheck/syntax) from validation the change breaks (a blocker).
- Do not stage artifacts validation creates (`__pycache__/`, `.pytest_cache/`,
  `node_modules/`, build caches).
- Push each commit in order:
  - if no upstream: `git push -u origin $(git branch --show-current)`
  - else: `git push`
- Retry transient network failures up to 4 times with exponential backoff
  (2s, 4s, 8s, 16s). Auth and permission failures are blockers to report, not
  failures to retry.
- Do not stop after the first group; the task ends when every group is pushed
  or a genuine blocker is hit. If a group fails mid-loop, report which commits
  were pushed and which were not — never silently continue or silently stop.

6. Open a draft PR only when explicitly requested.

- Do not open a PR automatically.
- Open only when both are true:
  - branch is not the default branch
  - user explicitly asks for a PR (or major-change PR)
- Prefer repo-native tooling; fallback to `gh pr create --draft`.

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
