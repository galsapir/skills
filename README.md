# claude-interview

A Claude Code plugin that provides the `/interview` command — a deep project interview that produces actionable specs before implementation begins.

## What it does

When you run `/interview` (optionally with a brief project description), Claude conducts a thorough requirements interview using structured questions. It adapts depth to project complexity, provides checkpoints where you can steer the conversation, and outputs a structured spec file or GitHub issue.

## Install

```
/plugin marketplace add galsapir/claude-interview
/plugin install claude-interview
```

## Usage

```
/interview                              # starts with "what are we building?"
/interview a CLI tool for managing dotfiles
/interview path/to/plan.md             # reads a plan file as starting point
```

## How it works

1. **Context gathering** — reads your project directory, git history, and CLAUDE.md for existing context
2. **Adaptive scoping** — estimates project size and tells you upfront how many questions to expect
3. **Structured interview** — asks 1-2 questions at a time, prefers multiple-choice, covers vision, UX, architecture, operations, and constraints
4. **Checkpoints every 3-5 questions** — you can continue at current depth, go higher level, skip to a topic, or wrap up
5. **Spec generation** — synthesizes answers into a structured spec with goals, non-goals, user stories, technical design, and edge cases
6. **Flexible output** — write to file, create a GitHub issue, both, or just display in chat

## Example

Here's a real session where `/interview` was used to spec out [Marginalia](https://github.com/galsapir/marginalia), a markdown annotation tool:

**Input:** "I want to create a single page application that would allow me to present a markdown file and annotate it with notes"

**Interview flow:**
- ~15 questions across 3 checkpoints
- Covered: annotation UX (text selection vs line-based), input method (paste vs file upload), persistence model, tech stack, UI layout, export format, edge cases
- User steered via checkpoints: "continue at this depth" for core features, wrapped up after edge cases

**Output:** A [172-line spec](https://github.com/galsapir/marginalia/blob/main/spec-markdown-notes.md) covering overview, goals/non-goals, user stories, technical design (component tree, data model, selection mapping algorithm), export format, edge cases, and open questions.

The spec was then used directly to build the full application in the same session.

## License

MIT
