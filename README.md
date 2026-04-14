# claude-skills

A collection of [Agent Skills](https://agentskills.io) by [Gal Sapir](https://github.com/galsapir).

Compatible with Claude Code, Cursor, GitHub Copilot, VS Code, Gemini CLI, and [30+ other agents](https://agentskills.io) that support the open Agent Skills standard.

## Install

```
npx skills add galsapir/claude-skills
```

This installs all skills from the repository. To install a specific skill:

```
npx skills add galsapir/claude-skills/interview
npx skills add galsapir/claude-skills/adversarial-review
```

## Skills

### `interview` — Project Interview

Deep project interview that produces actionable specs before implementation begins. Conducts a structured requirements interview with adaptive depth and checkpoints, then outputs a spec as a file, GitHub issue, or both.

```
/interview                                    # starts with "what are we building?"
/interview a CLI tool for managing dotfiles
/interview path/to/plan.md                   # reads a plan file as starting point
```

### `adversarial-review` — Independent Second Opinion

Gets an independent second opinion on code, specs, diffs, or GitHub issues from a separate AI model. Supports multiple reviewer backends for genuinely orthogonal perspectives.

```
/adversarial-review src/main.py                          # auto-detect backend
/adversarial-review src/main.py --backend codex          # use Codex (GPT family)
/adversarial-review src/main.py --backend claude          # use claude -p
/adversarial-review src/main.py --backend bedrock         # use AWS Bedrock
/adversarial-review diff                                  # review uncommitted changes
/adversarial-review #42                                   # review GitHub issue
/adversarial-review PR #7                                 # review pull request
/adversarial-review src/main.py --quick                   # skip Understanding section
/adversarial-review src/main.py --model gpt-5.4          # explicit model
```

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
