# Git Split Publish Skill (Claude)

When the user asks to split commits and publish, follow this core loop.
Operating stance: the publish request itself authorizes staging, committing,
and pushing what it covers — act on what the request already decides, ask once
(batched, with concrete options) about what it leaves genuinely open, finish
the whole loop, and lead the report with the outcome.

The full step-by-step procedure lives in `references/workflow.md`; the
commit-boundary rubric lives in `references/grouping-rules.md`.

## Core Loop

1. **Inspect git state** — evidence before any state change: `git status -sb`,
   `git diff --stat`, `git branch --show-current`, `git remote -v`. Before
   staging or committing for a publish request, `git remote get-url origin`
   and `git ls-remote origin` must succeed; if either fails, stop before
   staging or committing and report the blocker with the concrete next action.
   If the worktree is clean, report there is nothing to publish and stop.
2. **Read repository constraints** — root `AGENTS.md` if present, plus local
   branch/release/testing docs relevant to the changed files.
3. **Scope from the request** — do not re-ask what the user already decided.
   A whole-worktree request ("split all my changes and push") sets the scope; a
   request naming specific work publishes only that and reports what was left
   out. Never default to `git add -A`; use explicit path staging only.
4. **Group before staging** — one coherent task per commit; order groups so
   every commit builds/passes on its own; split cross-group hunks with
   `git add -p`/`git add -e` or commit once under the dominant intent and
   disclose the rider. Choose clearly defensible groupings and state the choice
   in the report; collect genuinely ambiguous files and ask about all of them
   in one question with concrete options — never group-by-group, never an
   open-ended "shall I proceed?".
5. **Commit and push one group at a time** — verify staged scope
   (`git diff --cached --stat`), validate before pushing and gate the push on
   the result, push each commit in order, retry transient network failures up
   to 4 times with exponential backoff (2s, 4s, 8s, 16s; auth/permission
   failures are blockers, not retries), and never stop after the first group —
   report which commits were pushed and which were not on a mid-loop failure.
6. **PR handling** — do not auto-create PRs; create a draft PR only when the
   user explicitly requests it (and the branch is not the default branch).

## Safety Rules

These hold regardless of how autonomous the run is:

- do not include unrelated changes silently
- do not discard or rewrite unstaged work
- do not amend/squash/reorder commits unless asked
- do not stage broadly (`git add -A`, `git add .`)
- screen every path before staging: never stage secrets or local env files
  (`.env`, `.env.*`, `*.pem`, `*.key`, credentials, tokens), dependency/build/
  cache artifacts (`node_modules/`, `__pycache__/`, `dist/`, `build/`, `.venv/`,
  `*.log`), or large/generated blobs (roughly >5 MB, or anything vendored or
  machine-generated); exclude them from the commit, flag them in the report, and
  if a secret looks already committed stop and warn instead of pushing; respect
  `.gitignore` and do not force-add ignored paths
- report blockers (missing remote/auth) with concrete next action before staging
  or committing

## Reporting

Lead with the outcome: the first sentence says what happened ("Pushed 3
commits to origin/main", "Stopped before committing: no push access to
origin"). Then cover, in readable prose rather than fragment checklists:

- branch/upstream state
- commit groups executed or proposed: hash, message, files, push status
- decisions made without asking, so the user can veto them after the fact
- what was intentionally left unstaged and why
- validation run and its result
- whether PR was skipped or created (and why)
- failures reported faithfully: include the actual error output; never claim
  success for anything unverified
