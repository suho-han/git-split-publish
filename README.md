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

## Manual Installation

### Codex

Install from this repository using your Codex skill installer flow, or clone
and place under:

- `~/.codex/skills/git-split-publish`

### Claude

Reference `CLAUDE.md` from your project-level instruction file (for example
`AGENTS.md`).

Example:

```md
When handling split-commit publish tasks, follow:
`<repo-root>/CLAUDE.md`
```

### Gemini

**Global Installation (Everywhere)**
Add a reference to this repository's `GEMINI.md` in your global
`~/.gemini/gemini.md` file:

```md
# Git Split Publish Skill
--- Context from: /absolute/path/to/git-split-publish/GEMINI.md ---
```

**Local Installation (Specific Project)**
Copy or symlink `GEMINI.md` to your project root, or include it in your
project's `GEMINI.md`:

```md
When handling split-commit publish tasks, follow:
`./GEMINI.md`
```

## Files

- `SKILL.md`: agent skill entry for all skill-compatible agents
- `CLAUDE.md`: Claude instruction block
- `GEMINI.md`: Gemini instruction block
- `AGENTS.md`: repository knowledge base (workflow summary, grouping rules, release policy)
- `references/grouping-rules.md`: commit grouping rubric
- `LICENSE`: MIT license

## Trigger Phrases

- split publish
- split changes by job and push
- group pending changes into separate commits and publish
