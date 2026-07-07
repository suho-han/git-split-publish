# Git Split Publish Skill (Gemini)

When the user asks to split commits and publish, follow this procedure. These instructions are foundational mandates. Operating stance: the publish request itself authorizes staging, committing, and pushing what it covers — act on what the request already decides, ask once (batched, with concrete options) about what it leaves genuinely open, finish the whole loop, and lead the report with the outcome.

## Procedure

1. **Inspect git state first — evidence before any state change.**
   - `git status -sb`
   - `git diff --stat`
   - `git branch --show-current`
   - `git remote -v`
   - If publish may happen: `git status --porcelain=v2 --branch`
   - Before staging or committing for publish, verify `git remote get-url origin`
     and `git ls-remote origin`; if either fails, stop before staging or
     committing and report the blocker with the concrete next action.
   - If worktree is clean, stop and report there is nothing to publish.

2. **Load repository guidance before grouping.**
   - Read root `AGENTS.md` or `GEMINI.md` if present in the target repository.
   - Read local docs for branch/release/validation policy when relevant.

3. **Determine scope from the request — do not re-ask what the user already decided.**
   - If the request covers the whole worktree ("split all my changes and push"),
     that is the scope; proceed without asking for confirmation.
   - If the request names specific work, publish only that, leave unrelated
     changes untouched, and report exactly what was left out.
   - Do not default to `git add -A`. Use explicit path staging: `git add -- <path>...`.

4. **Propose commit groups — decide what you can, batch what you cannot.**
   - Inspect per-file diffs when boundaries are unclear (`git diff -- <path>`);
     read the diff before asking about it.
   - Group by one coherent task per commit following `references/grouping-rules.md`.
   - For each group, provide: label, exact file list, commit message, and publish plan.
   - When one grouping is clearly defensible, choose it and state the choice in
     the report so the user can veto it after the fact.
   - Collect the genuinely ambiguous files — ones that plausibly belong to two
     groups where the choice changes what gets published — and ask about all of
     them in one question with concrete options. Never confirm group-by-group,
     and never ask an open-ended "shall I proceed?".

5. **Commit and push group-by-group — finish the whole loop.**
   - Stage each group's exact paths only.
   - Verify staged scope using `git diff --cached --stat`.
   - Commit with a terse, task-level message.
   - Run minimal relevant validation (e.g., lint/test) before the first push if applicable.
   - Push each commit in order:
     - If no upstream: `git push -u origin $(git branch --show-current)`
     - Else: `git push`
   - Retry transient network failures up to 4 times with exponential backoff
     (2s, 4s, 8s, 16s); auth/permission failures are blockers to report, not
     failures to retry.
   - Do not create a local commit for a publish request after discovering that
     the remote is missing, inaccessible, or unauthenticated.
   - Do not stop after the first group; if a group fails mid-loop, report which
     commits were pushed and which were not.

6. **Open draft PR only when explicitly requested.**
   - Do not open PR automatically.
   - Open only when: branch is not the default branch AND user explicitly asks for it.
   - Prefer repo-native tooling; fallback to `gh pr create --draft`.

## Guardrails

These hold regardless of how autonomous the run is:

- Do not silently include unrelated files.
- Do not rewrite or discard user changes.
- Do not amend, squash, or reorder existing commits unless requested.
- Do not stage broadly (`git add -A`, `git add .`).
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
