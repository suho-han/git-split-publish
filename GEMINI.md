# Git Split Publish Skill (Gemini)

When the user asks to split commits and publish, follow this core loop. These
instructions are foundational mandates. Operating stance: the publish request
itself authorizes staging, committing, and pushing what it covers — act on
what the request already decides, ask once (batched, with concrete options)
about what it leaves genuinely open, finish the whole loop, and lead the
report with the outcome.

The full step-by-step procedure lives in `references/workflow.md`; the
commit-boundary rubric lives in `references/grouping-rules.md`.

## Core Loop

1. **Inspect git state first** — evidence before any state change:
   `git status -sb`, `git diff --stat`, `git branch --show-current`,
   `git remote -v`. Before staging or committing for publish, verify
   `git remote get-url origin` and `git ls-remote origin`; if either fails,
   stop before staging or committing and report the blocker with the concrete
   next action. If the worktree is clean, stop and report there is nothing to
   publish.
2. **Load repository guidance before grouping** — root `AGENTS.md` (or
   `GEMINI.md`) if present in the target repository, plus local docs for
   branch/release/validation policy when relevant.
3. **Determine scope from the request** — do not re-ask what the user already
   decided. If the request covers the whole worktree ("split all my changes
   and push"), that is the scope; if it names specific work, publish only that
   and report exactly what was left out. Do not default to `git add -A`; use
   explicit path staging: `git add -- <path>...`.
4. **Propose commit groups** — decide what you can, batch what you cannot.
   Group by one coherent task per commit; order groups so every commit builds
   and passes on its own; split cross-group hunks with `git add -p` (or
   `git add -e` for contiguous hunks) or commit once under the dominant intent
   and disclose the rider. Choose clearly defensible groupings and state the
   choice in the report; collect genuinely ambiguous files and ask about all
   of them in one question with concrete options — never group-by-group, and
   never ask an open-ended "shall I proceed?".
5. **Commit and push group-by-group** — stage each group's exact paths only,
   verify staged scope (`git diff --cached --stat`), validate before pushing
   and gate the push on the result, push each commit in order (if no upstream:
   `git push -u origin $(git branch --show-current)`), retry transient network
   failures up to 4 times with exponential backoff (2s, 4s, 8s, 16s;
   auth/permission failures are blockers, not retries), and do not stop after
   the first group — report which commits were pushed and which were not on a
   mid-loop failure.
6. **Open draft PR only when explicitly requested** — do not open PRs
   automatically; open only when the branch is not the default branch AND the
   user explicitly asks. Prefer repo-native tooling; fallback to
   `gh pr create --draft`.

## Guardrails

These hold regardless of how autonomous the run is:

- Do not silently include unrelated files.
- Do not rewrite or discard user changes.
- Do not amend, squash, or reorder existing commits unless requested.
- Do not stage broadly (`git add -A`, `git add .`).
- Screen every path before staging. Never stage secrets or local env files
  (`.env`, `.env.*`, `*.pem`, `*.key`, credentials, tokens), dependency, build,
  or cache artifacts (`node_modules/`, `__pycache__/`, `dist/`, `build/`,
  `.venv/`, `*.log`), or large or machine-generated blobs (roughly >5 MB, or
  anything vendored or generated). Exclude such a file from the commit and flag
  it in the report; if a secret appears to be committed already, stop and warn.
  Respect `.gitignore` — do not force-add ignored paths.
- Stop before staging or committing and report concrete blockers (e.g., missing
  remote or auth) with clear next steps.

## Output Format

Lead with the outcome: the first sentence says what happened ("Pushed 3
commits to origin/main", "Stopped before committing: no push access to
origin"). Then cover, in readable prose rather than fragment checklists:

- Current branch and upstream state.
- Each commit group executed or proposed: hash, message, files, push result.
- Decisions made without asking, so the user can veto them after the fact.
- What was intentionally left unstaged and why.
- Validation results.
- PR decision (skipped or created) and the reason.
- Failures reported faithfully: include the actual error output; never claim
  success for anything unverified.
