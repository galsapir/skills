---
name: adversarial-review
description: >
  Get an independent second opinion from a separate AI model on code, specs,
  diffs, PRs, or GitHub issues. Use when the user asks for "second opinion",
  "adversarial review", "independent review", or "codex review". Supports
  subagent (fresh-context sub-Claude, default), interactive (tmux + codex),
  codex exec, claude -p, and AWS Bedrock backends.
  Arguments: [target] [--backend subagent|interactive|codex|claude|bedrock] [--model name] [--effort low|medium|high] [--quick]
license: MIT
compatibility: >
  `subagent` needs nothing beyond Claude Code itself. `interactive` needs tmux
  and codex CLI. `codex` / `claude` / `bedrock` need their respective CLIs or
  AWS credentials. Requires git.
allowed-tools: Bash(codex:*) Bash(claude:*) Bash(gh:*) Bash(git:*) Bash(uv:*) Bash(tmux:*) Read Grep Glob Agent
metadata:
  author: galsapir
  version: "1.2.0"
---

Independent second opinion from a separate AI model.

## Workflow

### 1. Detect Target

Parse `$ARGUMENTS`:

- File path(s) → code review
- `diff` / `changes` → `git diff` review
- `#N` or issue URL → `gh issue view` review
- PR URL or `PR #N` → `gh pr view` + `gh pr diff` review
- No args → ask user via AskUserQuestion

Extract flags: `--backend`, `--model`, `--effort` (codex only, default `medium`), `--quick`.

### 2. Reconnaissance

Gather context before building the review prompt:

1. Read the target (files, diff, issue, or PR)
2. Identify related files (imports, tests, config) and project context (README, CLAUDE.md)
3. Assemble a 2-5 sentence context summary: what the project does, what the target accomplishes, key related files

### 3. Select Backend

Priority: explicit `--backend` flag > auto-detect.

Auto-detect order: `subagent` (always available inside Claude Code — default) → `codex` (if `which codex`) → `bedrock` (if `scripts/bedrock-review.py` present and AWS creds) → `claude` (fallback).

`subagent` is the cheapest path — zero API keys, fresh context, no subprocess. Pick `interactive` when you want to drive the reviewer yourself (asking follow-ups, iterating). Pick `codex` / `claude` / `bedrock` when you need a specific external model.

Default models: codex=`gpt-5.4`, claude=`sonnet`, bedrock=`eu.anthropic.claude-sonnet-4-6-v1:0`. `subagent` runs on whatever model Claude Code is using. `interactive` uses codex defaults unless `--model` is passed. Override with `--model`.

### 4. Build Review Prompt

Read [references/review-prompt.md](references/review-prompt.md) and substitute placeholders:

- `{{CONTENT_TYPE}}` → code, spec, diff, issue, pr
- `{{REVIEW_TARGET}}` → file paths or inline content
- `{{CONTEXT}}` → project/architectural context from step 2
- `{{MODE}}` → `full` (default) or `quick` (if `--quick` flag or user said "delegate"/"just review")
- `{{GUIDANCE}}` → focus areas, related test files, key dependencies

Write assembled prompt to `/tmp/ar-prompt-$$.md`.

### 5. Execute Review

Capture git state before (`git diff --stat`), then run the chosen backend.

#### subagent (default)

Invoke the `Agent` tool directly:

- `subagent_type`: `general-purpose`
- `description`: `Adversarial review (<content_type>)`
- `prompt`: the full contents of `/tmp/ar-prompt-$$.md`

Use the subagent's returned text as the raw reviewer output for step 6. The subagent has its own tools (Read, Grep, Glob, Bash) and can explore the repo, but runs in a fresh context with no prior conversation.

#### interactive (tmux + codex)

Pre-reqs: must be running inside tmux (`tmux list-sessions` succeeds) and `codex` must be on PATH. If either fails, print a clear error and fall back to asking the user to pick another backend.

1. Launch a new window with codex reading the prompt via stdin:
   ```
   tmux new-window -n adv-review "codex < /tmp/ar-prompt-$$.md"
   ```
2. Tell the user: *"Opened tmux window `adv-review`. Drive the review to convergence, then in that window type: `output the final review JSON only`. When done, come back here and say `done` (or `cancel`)."*
3. Wait for the user to signal completion.
4. On `done`: capture the full pane buffer:
   ```
   tmux capture-pane -t adv-review -p -S - > /tmp/ar-capture-$$.txt
   ```
   Extract the last valid JSON object: pipe through `jq -R -s 'split("\n") | map(try fromjson // empty) | last'`. If that yields nothing, try extracting the last `{...}` block and validating with `jq .`. If extraction still fails, ask the user to paste the JSON directly.
5. Kill the window: `tmux kill-window -t adv-review`.
6. Feed the extracted JSON to step 6 as the raw reviewer output.

#### codex (non-interactive)

`codex exec "$(cat /tmp/ar-prompt-$$.md)" --sandbox read-only --model <model> -c model_reasoning_effort=<effort>` (use stdin for prompts >100KB; effort defaults to `medium`)

#### claude -p

`claude -p "$(cat /tmp/ar-prompt-$$.md)" --model <model> --output-format text`

#### bedrock

`uv run scripts/bedrock-review.py /tmp/ar-prompt-$$.md --model <model> --region eu-west-1`

---

After execution, run `git diff --stat && git status --short`. If any new changes appear, flag immediately.

### 6. Parse and Present Results

The reviewer returns a single JSON object (see prompt template). Strip any surrounding markdown code fences, then parse.

**On parse success**, render as markdown:

```
## Independent Review (via [backend] / [model])

**Verdict**: [SHIP | ITERATE | RETHINK]

[summary]

### Understanding
[understanding — omit this section entirely in quick mode or if null]

### Findings
[Each finding as its own block with every certificate field — severity, confidence, what, premises (each with evidence), trace, conclusion, alternative_hypothesis, suggestion, location. Do not drop premises, trace, or alternative_hypothesis for brevity.]

### Strengths
[omit section entirely if empty]

### Questions for the Author
[omit section entirely if empty]
```

Preserve every certificate field verbatim. Do NOT editorialize or soften. If you disagree with a finding, add a labeled `## Note from Claude` section *after* the review — never mutate the reviewer's output.

**On parse failure**, print the raw reviewer output under a warning:

```
## Independent Review (via [backend] / [model]) — raw output

> ⚠️ Reviewer did not return valid JSON. Output preserved verbatim below.

[raw text]
```

Do not attempt to synthesize findings from malformed output.

Clean up temp file.
