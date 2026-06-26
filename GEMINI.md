# Git Split Publish Skill (Gemini)

When the user asks to split commits and publish, follow this procedure strictly. These instructions are foundational mandates.

## Procedure

1. **Inspect git state first.**
   - `git status -sb`
   - `git diff --stat`
   - `git branch --show-current`
   - `git remote -v`
   - If publish may happen: `git status --porcelain=v2 --branch`
   - Before staging or committing for publish, verify `git remote get-url origin`
     and `git ls-remote origin`; if either fails, stop with a concrete blocker.
   - If worktree is clean, stop and report there is nothing to publish.

2. **Load repository guidance before grouping.**
   - Read root `AGENTS.md` or `GEMINI.md` if present in the target repository.
   - Read local docs for branch/release/validation policy when relevant.

3. **Propose commit groups.**
   - Inspect per-file diffs when boundaries are unclear (`git diff -- <path>`).
   - Group by one coherent task per commit following `references/grouping-rules.md`.
   - For each group, provide: label, exact file list, commit message, and publish plan.
   - If grouping is ambiguous, ask the user for clarification.

4. **Confirm scope before staging.**
   - On mixed worktrees, get explicit confirmation before staging.
   - Do not default to `git add -A`.
   - Use explicit path staging: `git add -- <path>...`.

5. **Commit and push group-by-group.**
   - Stage only approved files.
   - Verify staged scope using `git diff --cached --stat`.
   - Commit with a terse, task-level message.
   - Run minimal relevant validation (e.g., lint/test) before the first push if applicable.
   - Push each commit in order:
     - If no upstream: `git push -u origin $(git branch --show-current)`
     - Else: `git push`
   - Do not create a local commit for a publish request after discovering that
     the remote is missing, inaccessible, or unauthenticated.

6. **Open draft PR only when explicitly requested.**
   - Do not open PR automatically.
   - Open only when: branch is not the default branch AND user explicitly asks for it.
   - Prefer repo-native tooling; fallback to `gh pr create --draft`.

## Guardrails

- Do not silently include unrelated files.
- Do not rewrite or discard user changes.
- Do not amend, squash, or reorder existing commits unless requested.
- Stop before staging or committing and report concrete blockers (e.g., missing
  remote or auth) with clear next steps.

## Output Format

Report the following after execution:
- Current branch and upstream state.
- Proposed or executed commit groups.
- Created commit hashes and messages.
- Push result per commit.
- PR decision (skipped or created) and the reason.
- Validation results and any remaining blockers.
