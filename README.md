# claude-skills

Personal Claude Code skills collection by [Gal Sapir](https://github.com/galsapir).

## Skills

### `/interview` — Project Interview

Deep project interview that produces actionable specs before implementation begins. Conducts a structured requirements interview with adaptive depth and checkpoints, then outputs a spec as a file, GitHub issue, or both.

```
/interview                                    # starts with "what are we building?"
/interview a CLI tool for managing dotfiles
/interview path/to/plan.md                   # reads a plan file as starting point
```

### `/adversarial-review` — Independent Second Opinion

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
/adversarial-review src/main.py --model gpt-5.3-codex    # explicit model
```

**Backends**:

| Backend | Models | Notes |
|---------|--------|-------|
| `codex` | `o4-mini` (default), `gpt-5.3-codex`, `gpt-5.2-codex` | Most orthogonal — different model family |
| `claude` | `sonnet` (default), any Claude Code model | Same family, fresh context |
| `bedrock` | Claude Sonnet 4.5 (default), Haiku 4.5 | Same family via AWS; extensible to Llama/Nova/Mistral |

**Output structure**: Executive Summary (SHIP/ITERATE/RETHINK verdict), Understanding (full mode), Findings (severity + confidence rated), Strengths, Questions for Author.

## Install

```
/install-plugin galsapir/claude-skills
```

## Prerequisites

- **Codex backend**: `npm install -g @openai/codex` + `codex auth`
- **Bedrock backend**: AWS credentials configured with Bedrock access in `eu-west-1`
- **Claude backend**: Claude Code CLI (you already have this)

## License

MIT
