---
name: interview
description: >
  Deep project interview that produces actionable specs before implementation begins.
  Conducts a structured requirements interview with adaptive depth and checkpoints,
  then outputs a spec as a file, GitHub issue, or both. When given an existing plan
  or design, switches to grill mode — stress-testing decisions and walking the design
  tree. Use when the user wants to scope a project, plan a feature, write a spec,
  stress-test a design, or says "interview" or "grill me".
license: MIT
compatibility: Designed for Claude Code (or similar products)
metadata:
  author: galsapir
  version: "1.1.0"
  credits: grill mode inspired by mattpocock/skills (MIT)
  model-hint: opus
---

Conduct a thorough project interview to produce a clear, actionable spec before any implementation begins.

## Context Gathering

First, gather context about the current state:

1. Check if this is an existing project or greenfield:
   - Read the current directory structure (use `ls` and check for package.json, pyproject.toml, go.mod, CLAUDE.md, README, etc.)
   - If a git repo exists, check recent commits for context
   - If a CLAUDE.md exists, read it for project conventions
2. If `$ARGUMENTS` references a file path, read that file as the initial plan/idea
3. If `$ARGUMENTS` is a text description, use it as the starting point
4. If no arguments provided, ask what the project/feature is about

## Mode Selection

After gathering context, determine the interview mode:

**Grill mode** — activate when ANY of these are true:
- The user provides an existing plan, design doc, spec, or architecture
- `$ARGUMENTS` references a file containing a plan or design
- The user says "grill me", "stress-test", "poke holes", or similar
- There's clearly an existing design to validate rather than requirements to discover

**Discovery mode** — activate when the user is starting from scratch or has only a vague idea.

State which mode you're using at the start: "You've got an existing plan, so I'll switch to grill mode — I'll stress-test your decisions and walk through the design tree." or "Starting from scratch, so I'll run a discovery interview to uncover requirements."

---

## Discovery Mode

Interview the user about their project using AskUserQuestion. The goal is to uncover requirements, constraints, edge cases, and design decisions that the user hasn't thought through yet.

### Question Design Principles

- Ask non-obvious questions. Don't ask things the user has already stated or that are self-evident.
- Probe for hidden complexity, unstated assumptions, and edge cases.
- Adapt question depth to project complexity — a weekend hack needs 10 questions, not 75.
- Prefer multiple-choice options when possible (with "Other" always available), but use open-ended questions when the answer space is too broad for predefined options.
- Ask 1-2 questions per AskUserQuestion call. Group related questions together.
- Frame questions around concrete scenarios ("What happens when a user tries to X while Y is happening?") rather than abstract concepts.

### Question Categories

Cover these areas as relevant to the project. Skip categories that clearly don't apply:

**Core Vision & Scope**
- What problem does this solve? Who is it for?
- What's the MVP vs nice-to-have?
- What does "done" look like?

**User Experience & Behavior**
- Key user flows and interactions
- Error states and edge cases from the user's perspective
- What happens when things go wrong?

**Technical Architecture**
- Tech stack choices (if not already decided)
- Data model and storage
- External dependencies and integrations
- Performance requirements and constraints

**Operational Concerns**
- Deployment and infrastructure
- Monitoring, logging, error handling
- Security considerations

**Project Constraints**
- Timeline and resource constraints
- Existing code/systems to integrate with
- Hard requirements vs flexible preferences

### Checkpoint Mechanism

After every 3-5 questions, present a checkpoint using AskUserQuestion:

```
Question: "Checkpoint: I've covered [areas covered so far]. Remaining areas to explore: [remaining areas]. How should we proceed?"
Options:
- "Continue at this depth" — keep going with detailed questions
- "Go higher level" — switch to broader, fewer questions for remaining areas
- "Skip to [specific area]" — jump to a particular topic
- "Wrap up" — enough info, generate the spec now
```

At each checkpoint, briefly summarize what has been established so far (2-3 sentences max) before asking how to proceed.

If the user says "wrap up" or "go higher level", respect that immediately. Don't sneak in extra detailed questions.

### Adaptive Depth

Gauge project complexity from the initial description and context:

- **Small feature/bugfix**: 5-10 questions, 1-2 checkpoints
- **Medium feature**: 10-20 questions, 2-3 checkpoints
- **Large project/system**: 20-40 questions, 4-6 checkpoints
- **Greenfield application**: 30-50+ questions, 5-8 checkpoints

State the estimated scope at the start: "This seems like a [size] project. I'll aim for roughly [N] questions across [M] checkpoints. We can always adjust."

---

## Grill Mode

Stress-test the user's plan or design by walking through every branch of the decision tree. The goal is to reach shared understanding — every assumption surfaced, every dependency resolved, every edge case considered.

### Approach

- **Be adversarial.** Your job is to find holes, contradictions, and unstated assumptions. Don't accept hand-wavy answers.
- **Walk the decision tree.** Identify the major design decisions, then drill into each branch sequentially. Resolve dependencies between decisions before moving on.
- **One question at a time.** Each question should target a single decision point. Don't batch questions — let the user think deeply about each one.
- **Explore the codebase first.** If a question can be answered by reading existing code, read the code instead of asking. Only ask when the answer requires human judgment or domain knowledge.
- **No suggested answers by default.** Let the user think freely. If the user explicitly asks for suggestions or recommendations, then provide your opinion with each question going forward.
- **Frame around consequences.** "What happens when X fails?" is better than "Have you thought about X?"

### Decision Tree Traversal

1. Read the plan/design thoroughly
2. Identify the top-level decisions and assumptions
3. For each decision:
   - Ask why this choice was made over alternatives
   - Probe the implications and edge cases of the choice
   - Identify dependencies on other decisions
   - If a dependency is unresolved, switch to that branch first
4. Track which branches are resolved vs open
5. Continue until all branches are resolved or the user wraps up

### Checkpoint Mechanism

After resolving 3-5 decision branches, checkpoint:

```
Question: "Checkpoint: I've stress-tested [resolved areas]. Open branches: [remaining]. How should we proceed?"
Options:
- "Keep grilling" — continue with remaining branches
- "Skip to [specific area]" — jump to a particular decision
- "Wrap up" — enough, generate the findings
```

## Output

When the interview is complete (either all areas covered or user says "wrap up"):

### Discovery Mode → Spec

1. Synthesize all answers into a structured spec document
2. Use this format:

```markdown
# [Project/Feature Name] — Spec

## Overview
[2-3 sentence summary of what this is and why it exists]

## Goals & Non-Goals
### Goals
- [Concrete, measurable goals]

### Non-Goals
- [Explicitly out of scope items]

## User Stories / Key Flows
- As a [who], I want to [what], so that [why]
- [Key interaction flows described concretely]

## Technical Design
### Architecture
[High-level architecture decisions]

### Data Model
[Key entities and relationships, if applicable]

### API / Interface Design
[Endpoints, function signatures, or UI components, if applicable]

### Dependencies
[External services, libraries, tools]

## Edge Cases & Error Handling
- [Identified edge cases and how to handle them]

## Open Questions
- [Anything that came up during the interview that wasn't fully resolved]

## Implementation Notes
- [Key technical decisions, constraints, or preferences from the interview]
```

3. Adapt the template — skip sections that don't apply, add sections that do.

### Grill Mode → Findings Report

1. Synthesize the stress-test into a findings report
2. Use this format:

```markdown
# [Plan/Design Name] — Grill Report

## Summary
[2-3 sentence verdict: is the plan solid, needs work, or has fundamental issues?]

## Resolved Decisions
- **[Decision]**: [What was decided and why it holds up]

## Issues Found
### Critical
- [Issues that would block or break the plan]

### Gaps
- [Missing details that need answers before implementation]

### Risks
- [Things that could go wrong but have mitigation paths]

## Assumptions Surfaced
- [Implicit assumptions that were made explicit during the grill]

## Recommendations
- [Concrete next steps to address issues and gaps]
```

3. Adapt the template — skip sections that don't apply.

## Output Delivery

After generating the spec, ask the user with AskUserQuestion:

```
Question: "Spec is ready. Where should I put it?"
Options:
- "Write to file" — write spec to a file in the current directory (will ask for filename)
- "Create GitHub issue" — create a GH issue with the spec content
- "Both" — file + GH issue
- "Just show me" — display in chat, don't write anywhere
```

If writing to file: suggest a filename like `spec-[project-name].md` and confirm before writing.
If creating GH issue: use `gh issue create` with the spec as the body. Ask for labels/milestone if the repo uses them.

## Important

- This is an INTERVIEW, not a lecture. Listen more than you talk.
- In discovery mode: don't inject your own architectural opinions unless asked. Surface tradeoffs and let the user decide.
- In grill mode: be opinionated about problems you find, but don't suggest answers unless the user asks for them.
- If the user gives a one-word answer, that's fine. Don't push for more unless the answer is genuinely ambiguous.
- Keep checkpoint summaries SHORT. The user doesn't want to re-read everything they just told you.
- Never start implementing. This command produces a spec or findings report, nothing more.
