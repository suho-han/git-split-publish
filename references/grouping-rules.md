# Commit Grouping Rules

Use these rules when a worktree contains multiple logical tasks.

1. One intent per commit.
- Keep each commit focused on one task, fix, or docs-only change.

2. Separate risk levels.
- Keep behavior-changing code separate from refactor or formatting-only edits.

3. Separate runtime and tooling.
- Keep build/tooling config changes separate from product code when possible.

4. Keep tests with their target change.
- Include tests with the behavior they verify, unless tests are shared across multiple groups.

5. Keep generated artifacts separate unless policy requires coupling.
- If the repo versions generated output, include only artifacts caused by that group.

6. Each commit stands on its own.
- Prefer commits that build and pass their checks independently, so history
  stays bisectable.
- When several changes share a new symbol or helper, give that shared code its
  own foundational commit ordered first (or fold it into the earliest change
  that needs it) so no commit references something a later commit introduces.
- Ordering is what makes this hold; a worktree-level check does not prove it,
  because later groups are still present when it runs. When it must be
  guaranteed, validate the commit in isolation before pushing — build its
  committed tree (`git archive <commit>` into a temp dir) or `git stash -u` the
  later groups so the check sees only that commit.

7. Split a file whose hunks span groups.
- One file can carry changes for two intents (for example a manifest that adds a
  runtime dependency for a feature and a test script for tooling).
- Separate them with `git add -p` (or `git add -e` for contiguous hunks that
  patch mode refuses to split). If they truly cannot be separated, commit the
  file once under its dominant intent and disclose the rider hunk in the report.

8. Never fold in secrets, junk, or bulk.
- Do not stage secrets or local env files (`.env`, `.env.*`, `*.pem`, `*.key`,
  credentials, tokens), dependency/build/cache artifacts (`node_modules/`,
  `__pycache__/`, `dist/`, `build/`, `.venv/`, `*.log`), or large or generated
  blobs (roughly >5 MB, or anything vendored or machine-generated).
- Exclude such a file from every group and flag it in the report; if a secret
  looks already committed, stop and warn.

9. Decide when defensible, ask once when not.
- Read the file's diff before treating its grouping as ambiguous; untracked
  files show nothing under `git diff -- <path>`, so inspect them with
  `git diff --no-index -- /dev/null <path>` or by reading the file.
- If one grouping is clearly defensible, choose it and state the choice in the
  final report so the user can veto it after the fact.
- Treat a file as genuinely ambiguous only when it plausibly belongs to two
  groups and the choice changes what gets published.
- Collect every genuinely ambiguous file and ask about all of them in a single
  question with concrete options, before staging — never one question per file.
