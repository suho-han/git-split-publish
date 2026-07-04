# Git Split Publish Knowledge Base

## Overview

This repo is a portable assistant skill package for safely splitting a mixed
git worktree into task-based commits, pushing each commit in order, and opening
a draft PR only when explicitly requested. It is documentation-only: no source
tree, runtime package, CI, build, lint, test, or release automation exists.

## Files

| File | Purpose |
|------|---------|
| `README.md` | Package index, install paths, trigger phrases. |
| `SKILL.md` | Codex skill entry; version `0.1.0`, author `suhohan`, homepage `https://github.com/suho-han/git-split-publish`. |
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

1. Inspect state first:
   - `git status -sb`
   - `git diff --stat`
   - `git branch --show-current`
   - `git remote -v`
   - if push may happen: `git status --porcelain=v2 --branch`
   - if clean: stop and report nothing to publish
2. Read local constraints before grouping:
   - root `AGENTS.md` if present
   - branch/release/validation docs relevant to changed files
3. Propose groups before staging:
   - one coherent task per commit
   - inspect unclear boundaries with `git diff -- <path>`
   - report label, exact files, commit message, and publish/PR intent
4. Confirm scope on mixed worktrees:
   - never assume the full worktree is in scope
   - never use blanket `git add -A`
   - stage explicit paths only: `git add -- <path>...`
5. Commit and push one group at a time:
   - verify staged scope with `git diff --cached --stat`
   - use `git diff --cached` when the stat is not enough
   - run minimal relevant validation before the first push
   - push in order; if no upstream, use `git push -u origin $(git branch --show-current)`
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
- Ask before staging if one file plausibly belongs to multiple groups.

## Do Not

- Do not silently include unrelated files.
- Do not discard, rewrite, amend, squash, or reorder user work unless requested.
- Do not stage broadly.
- Do not invent build/test/lint/release commands for this repo.
- Do not add nested `AGENTS.md` under `references/` or `.github/` while they are
  single-file support directories.

## Response Contract

Always include:

- branch/upstream state
- commit groups proposed or executed
- commit hashes and push status
- PR skipped/created decision and reason
- validation run and remaining blockers

## Notes

- Remote: `origin` -> `https://github.com/suho-han/git-split-publish.git`.
- Release work means keeping `SKILL.md`, `CLAUDE.md`, `GEMINI.md`, and
  `references/grouping-rules.md` semantically aligned.
