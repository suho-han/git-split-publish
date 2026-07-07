# Git Split Publish Skill (Claude)

When the user asks to split commits and publish, follow this procedure. Operating stance: the publish request itself authorizes staging, committing, and pushing what it covers — act on what the request already decides, ask once (batched, with concrete options) about what it leaves genuinely open, finish the whole loop, and lead the report with the outcome.

## Procedure

1. Inspect git state — evidence before any state change.
- `git status -sb`
- `git diff --stat`
- `git branch --show-current`
- `git remote -v`
- if publish may happen: `git status --porcelain=v2 --branch`
- before staging/committing for publish: verify `git remote get-url origin`
  and `git ls-remote origin`; if either fails, stop before staging or
  committing and report the blocker with the concrete next action
- if the worktree is clean, report there is nothing to publish and stop

2. Read repository constraints.
- read `AGENTS.md` (if present)
- read branch/release/testing docs relevant to changed files

3. Determine scope from the request — do not re-ask what the user already decided.
- if the request covers the whole worktree ("split all my changes and push"),
  that is the scope; proceed without asking for confirmation
- if the request names specific work, publish only that, leave unrelated
  changes untouched, and report exactly what was left out
- never default to `git add -A`; use explicit path staging only

4. Group — decide what you can, batch what you cannot.
- group by one coherent task per commit (see `references/grouping-rules.md`)
- inspect per-file diffs when unclear (`git diff -- <path>`); read the diff
  before asking about it
- for each group: label, exact file list, commit message, publish plan
- when one grouping is clearly defensible, choose it and state the choice in
  the report so the user can veto it after the fact
- collect the genuinely ambiguous files — ones that plausibly belong to two
  groups where the choice changes what gets published — and ask about all of
  them in one question with concrete options; never confirm group-by-group,
  never ask an open-ended "shall I proceed?"

5. Commit and push one group at a time — finish the whole loop.
- verify staged diff (`git diff --cached --stat`)
- commit with concise message
- run minimal relevant validation before the first push
- push each commit in order
- if no upstream: `git push -u origin $(git branch --show-current)`
- retry transient network failures up to 4 times with exponential backoff
  (2s, 4s, 8s, 16s); auth/permission failures are blockers to report, not
  failures to retry
- never create a local commit for a publish request after remote/auth has
  already failed; report the blocker instead
- do not stop after the first group; if a group fails mid-loop, report which
  commits were pushed and which were not

6. PR handling.
- do not auto-create PR
- create draft PR only when user explicitly requests it

## Safety Rules

These hold regardless of how autonomous the run is:

- do not include unrelated changes silently
- do not discard or rewrite unstaged work
- do not amend/squash/reorder commits unless asked
- do not stage broadly (`git add -A`, `git add .`)
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
