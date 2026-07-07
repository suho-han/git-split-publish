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

6. Decide when defensible, ask once when not.
- Read the file's diff before treating its grouping as ambiguous.
- If one grouping is clearly defensible, choose it and state the choice in the
  final report so the user can veto it after the fact.
- Treat a file as genuinely ambiguous only when it plausibly belongs to two
  groups and the choice changes what gets published.
- Collect every genuinely ambiguous file and ask about all of them in a single
  question with concrete options, before staging — never one question per file.
