# git-split-publish

Portable `git-split-publish` skill package for Codex, Claude, Gemini, and other
skill-compatible agents.

## Install

Git and your chosen agent must already be installed. Run the commands below
in your terminal to install for your user account. Clone the whole repository
so `SKILL.md` and `references/` stay together.

### Codex

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/suho-han/git-split-publish.git ~/.agents/skills/git-split-publish
```

Start a new Codex session and invoke `$git-split-publish`, or use one of the
trigger phrases below.
See the [Codex skills documentation](https://developers.openai.com/codex/skills).

### Claude Code

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/suho-han/git-split-publish.git ~/.claude/skills/git-split-publish
```

Start a new Claude Code session and invoke `/git-split-publish`, or use one
of the trigger phrases below.
See the [Claude Code skills documentation](https://code.claude.com/docs/en/skills).

### Gemini CLI

```bash
mkdir -p ~/.gemini/skills
git clone https://github.com/suho-han/git-split-publish.git ~/.gemini/skills/git-split-publish
```

Run `/skills reload`, then `/skills list` in Gemini CLI to check that
`git-split-publish` is available. Use one of the trigger phrases below.
See the [Gemini CLI skills documentation](https://geminicli.com/docs/cli/skills/).

### Other skill-compatible agents

Copy or clone the whole repository into the skills directory documented by
your agent, with `SKILL.md` directly inside `git-split-publish/`.

### Update an existing installation

If the destination already exists, update its clone instead of cloning again.
Use the path for your agent, for example:

```bash
git -C ~/.agents/skills/git-split-publish pull --ff-only
```

Start a new agent session or reload its skills after updating.

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
