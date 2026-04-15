# claude-skills

A collection of [Agent Skills](https://agentskills.io) by [Gal Sapir](https://github.com/galsapir).

Compatible with Claude Code, Cursor, GitHub Copilot, VS Code, Gemini CLI, and [many other agents](https://agentskills.io) that support the open Agent Skills standard.

## Install

```
npx skills add galsapir/claude-skills
```

This installs all skills from the repository. To install a specific skill:

```
npx skills add galsapir/claude-skills --skill interview
npx skills add galsapir/claude-skills --skill adversarial-review
```

## Skills

Skills are invoked by describing the task in natural language — the agent selects a matching skill based on its description. No slash command needed.

### `interview` — Project Interview

Deep project interview that produces actionable specs before implementation begins. Conducts a structured requirements interview with adaptive depth and checkpoints, then outputs a spec as a file, GitHub issue, or both.

```
"use the interview skill to scope a CLI tool for managing dotfiles"
"interview me about adding dark mode to settings"
"use the interview skill with this plan: path/to/plan.md"
```

### `adversarial-review` — Independent Second Opinion

Gets an independent second opinion on code, specs, diffs, or GitHub issues from a separate AI model. Supports multiple reviewer backends for genuinely orthogonal perspectives.

```
"run adversarial-review on src/main.py"
"adversarial-review src/main.py with the codex backend"
"adversarial-review my uncommitted changes"
"adversarial-review issue #42"
"adversarial-review PR #7"
"adversarial-review src/main.py (quick mode)"
```

The skill accepts these arguments: `[target] [--backend codex|claude|bedrock] [--model name] [--quick]`.

**Backends**:

| Backend | Default Model | Notes |
|---------|---------------|-------|
| `codex` | `gpt-5.4` | Most orthogonal — different model family |
| `claude` | `sonnet` | Same family, fresh context |
| `bedrock` | `claude-sonnet-4-6` | Same family via AWS; extensible to Llama/Nova/Mistral |

**Output structure**: Executive Summary (SHIP/ITERATE/RETHINK verdict), Understanding (full mode), Findings (severity + confidence rated), Strengths, Questions for Author.

## Repository Structure

This repository follows the [Agent Skills specification](https://agentskills.io/specification):

```
skills/
  interview/
    SKILL.md              # Skill metadata + instructions
  adversarial-review/
    SKILL.md              # Skill metadata + instructions
    references/
      review-prompt.md    # Review prompt template
    scripts/
      bedrock-review.py   # AWS Bedrock backend script
```

Each skill is a self-contained directory with a `SKILL.md` file containing YAML frontmatter (`name`, `description`, and optional fields) followed by Markdown instructions.

## Migrating from the Plugin Marketplace

If you previously installed using the old Claude Code plugin marketplace format:

```
/plugin uninstall gal-skills
```

Then reinstall using the skills CLI:

```
npx skills add galsapir/claude-skills
```

## Prerequisites

- **Codex backend**: `npm install -g @openai/codex` + `codex auth`
- **Bedrock backend**: AWS credentials configured with Bedrock access in `eu-west-1`
- **Claude backend**: Claude Code CLI (you already have this)

## License

MIT
