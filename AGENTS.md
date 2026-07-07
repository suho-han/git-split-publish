# Git Split Publish Knowledge Base

## Overview

This repo is a portable assistant skill package for safely splitting a mixed
git worktree into task-based commits, pushing each commit in order, and opening
a draft PR only when explicitly requested. It is documentation-only: no source
tree, runtime package, CI, build, lint, test, or release automation exists.

## Operating Philosophy

The skill runs on calibrated autonomy rather than blanket confirmation:

- The publish request itself authorizes staging, committing, and pushing the
  changes it covers. Act on what the request already decides; do not re-ask it.
- Reserve questions for decisions the request leaves genuinely open, and ask
  them once, batched, with concrete options — never group-by-group, never an
  open-ended "shall I proceed?".
- Finish the whole loop: every group committed and pushed, or a genuine
  blocker reported. Retry transient network failures; never retry auth.
- Lead every report with the outcome, state decisions made without asking so
  the user can veto them, and report failures with the actual error output.

## Files

| File | Purpose |
|------|---------|
| `README.md` | Package index, install paths, trigger phrases. |
| `SKILL.md` | Codex skill entry; version `0.2.0`, author `suhohan`, homepage `https://github.com/suho-han/git-split-publish`. |
| `CLAUDE.md` | Claude-facing copy of the workflow. |
| `GEMINI.md` | Gemini-facing copy; references grouping rules. |
| `references/grouping-rules.md` | Commit boundary policy. |
| `.github/FUNDING.yml` | Sponsorship metadata only. |

## Installation / Triggers

- Codex skill path: `~/.codex/skills/git-split-publish`.
- Claude: reference `CLAUDE.md` from a project instruction file.
- Gemini: reference or copy/symlink `GEMINI.md`.
- Trigger phrases: "split publish", "split changes by job and push",
  "group pending changes into separate commits and publish".

## Required Workflow

1. Inspect state first — evidence before any state change:
   - `git status -sb`
   - `git diff --stat`
   - `git branch --show-current`
   - `git remote -v`
   - if push may happen: `git status --porcelain=v2 --branch`
   - before staging/committing for publish: `git remote get-url origin` and
     `git ls-remote origin` must succeed, else stop and report the blocker
     with the concrete next action
   - if clean: stop and report nothing to publish
2. Read local constraints before grouping:
   - root `AGENTS.md` if present
   - branch/release/validation docs relevant to changed files
3. Determine scope from the request:
   - a whole-worktree request ("split all my changes and push") sets the scope;
     proceed without asking for confirmation
   - a request naming specific work publishes only that; leave unrelated
     changes untouched and report what was left out
   - never use blanket `git add -A`; stage explicit paths only:
     `git add -- <path>...`
4. Group before staging — decide what you can, batch what you cannot:
   - one coherent task per commit
   - inspect unclear boundaries with `git diff -- <path>` before asking;
     untracked files show nothing there, so use
     `git diff --no-index -- /dev/null <path>` or read the file
   - order groups so every commit builds/passes on its own: shared new
     symbols/helpers get their own foundational commit first (or ride with the
     earliest change that needs them) so no commit references a later addition
   - when one file's hunks span groups, split with `git add -p`/`git add -e`;
     if inseparable, commit once under the dominant intent and disclose the rider
   - report label, exact files, commit message, and publish/PR intent per group
   - choose clearly defensible groupings and state the choice in the report;
     batch the genuinely ambiguous files into one question with concrete options
5. Commit and push one group at a time — finish the whole loop:
   - verify staged scope with `git diff --cached --stat`
   - use `git diff --cached` when the stat is not enough
   - run minimal relevant validation before the first push and gate the push on
     it: if the change breaks validation, stop and report the actual output;
     distinguish validation that cannot run (missing deps/broken harness —
     caveat, fall back to build/typecheck/syntax) from validation the change
     breaks (blocker); do not stage artifacts validation creates (`__pycache__/`,
     `.pytest_cache/`, `node_modules/`, build caches)
   - push in order; if no upstream, use `git push -u origin $(git branch --show-current)`
   - retry transient network failures up to 4 times with exponential backoff
     (2s, 4s, 8s, 16s); auth/permission failures are blockers, not retries
   - never create a local commit after remote/auth has already failed
   - do not stop after the first group; on mid-loop failure, report which
     commits were pushed and which were not
6. PRs:
   - do not auto-create PRs
   - create a draft PR only if the user asks and the branch is not default
   - prefer repo-native tooling; fallback: `gh pr create --draft`

## Grouping Rules

- One intent per commit.
- Separate behavior changes from refactors or formatting-only edits.
- Separate tooling/config from runtime/product changes when possible.
- Keep tests with the behavior they verify.
- Keep generated artifacts separate unless repo policy requires coupling.
- Keep history bisectable: each commit should build/pass on its own, and shared
  new symbols belong in a foundational commit ordered before their dependents.
- When a single file's hunks span groups, split with `git add -p`/`git add -e`,
  or commit once under the dominant intent and disclose the rider.
- Never fold in secrets, env files, dependency/build/cache artifacts, or large
  generated blobs; exclude and flag them instead.
- Decide clearly defensible groupings and state the choice in the report; ask
  only about files that plausibly belong to two groups where the choice
  changes what gets published, batched into a single question before staging.

## Do Not

- Do not silently include unrelated files.
- Do not discard, rewrite, amend, squash, or reorder user work unless requested.
- Do not stage broadly (`git add -A`, `git add .`).
- Do not stage secrets or local env files (`.env`, `.env.*`, `*.pem`, `*.key`,
  credentials, tokens), dependency/build/cache artifacts (`node_modules/`,
  `__pycache__/`, `dist/`, `build/`, `.venv/`, `*.log`), or large/generated
  blobs (roughly >5 MB, or anything vendored or machine-generated); exclude and
  flag them, and if a secret looks already committed, stop and warn. Respect
  `.gitignore` and do not force-add ignored paths.
- Do not re-ask scope the request already decided, and do not ask open-ended
  "shall I proceed?" questions.
- Do not claim success for anything unverified.
- Do not invent build/test/lint/release commands for this repo.
- Do not add nested `AGENTS.md` under `references/` or `.github/` while they are
  single-file support directories.

## Response Contract

Lead with the outcome — the first sentence says what happened. Then always
include:

- branch/upstream state
- commit groups proposed or executed
- commit hashes and push status
- decisions made without asking, so the user can veto them
- what was intentionally left unstaged and why
- PR skipped/created decision and reason
- validation run and remaining blockers, with actual error output on failure

## Notes

- Remote: `origin` -> `https://github.com/suho-han/git-split-publish.git`.
- Release work means keeping `SKILL.md`, `CLAUDE.md`, `GEMINI.md`, and
  `references/grouping-rules.md` semantically aligned.
