# git-split-publish

Portable `git-split-publish` skill package for Codex, Claude, Gemini, and other
skill-compatible agents.

## Install (Smithery)

This skill is listed on the Smithery Skills Registry:

- <https://smithery.ai/skills/suho-han/git-split-publish>

Install with the Skills CLI — it detects your agents and installs for all of
them (Claude Code, Cursor, Codex, Windsurf, Cline, Goose, Gemini CLI, and
more):

```bash
npx skills add suho-han/git-split-publish
```

Or install for a specific agent with the Smithery CLI:

```bash
npm install -g smithery
smithery skill add suho-han/git-split-publish --agent claude-code
```

Per-platform commands are also shown on the
[Smithery skill page](https://smithery.ai/skills/suho-han/git-split-publish).

Prefer a manual setup? Copy or clone this repository into your agent's skills
directory (for example `~/.claude/skills/git-split-publish`,
`~/.codex/skills/git-split-publish`, or the universal
`~/.agents/skills/git-split-publish`).

## Files

- `SKILL.md`: agent skill entry for all skill-compatible agents
- `AGENTS.md`: repository knowledge base (workflow summary, grouping rules, release policy)
- `references/workflow.md`: full step-by-step publish workflow
- `references/grouping-rules.md`: commit grouping rubric
- `LICENSE`: MIT license

## Trigger Phrases

- split publish
- split changes by job and push
- group pending changes into separate commits and publish
