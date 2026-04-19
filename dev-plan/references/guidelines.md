# Agent Dev Plan — Reference

## 1) Required `plan.md` outline

`plan.md` should stay short, skimmable, and useful to both humans and agents.

Include these sections when creating or substantially rewriting the plan:

1. `Task`
2. `Context`
3. `Assumptions` (omit if none)
4. `Open Questions` (omit if none)
5. `Plan`
6. `Verification`
7. `Decisions` (omit if no DRs are relevant yet)

Guidance:
- Use a task reference, not a tracker-specific label. Examples: Jira ticket, Linear issue, GitHub issue, design doc, or local task name.
- Keep facts in `Context`, guesses in `Assumptions`, and unresolved items in `Open Questions`.
- Use 3-6 plan checkpoints by default.
- Sub-bullets are optional. Add them only when they carry real value such as dependencies, risks, or notes.
- Always include at least one verification step.
- Keep the plan human-readable. Avoid filler items added only to satisfy structure.

Recommended template:

```md
# Task Plan

## Task
- Reference: <task reference or local description>
- Goal: <what success looks like>

## Context
- <facts from prompt, docs, codebase, issue tracker>

## Assumptions
- <non-blocking assumptions>

## Open Questions
- <open items that could still change the plan>

## Plan
- [ ] 1. <checkpoint title>
  - Notes / risks / dependencies: <optional>
- [ ] 2. <checkpoint title>
- [ ] 3. <checkpoint title>

## Verification
- [ ] Validate behavior against acceptance criteria
- [ ] Run or describe relevant tests / checks

## Decisions
- DR-001, DR-002
```

## 2) Decision Record format (`.plan/<branch>/decisions.md`)

Each decision should use this template:

## DR-### — <Decision Title>
- Date: YYYY-MM-DD
- Branch: <branch>
- Status: proposed | accepted | superseded
- Context: what problem were we solving?
- Decision: what did we choose?
- Why: constraints and reasoning
- Alternatives considered: options and why not
- Tradeoffs: gains vs costs
- Metrics: measurements given, if any
- Consequences: what this enables or complicates
- Links: PR/issue/doc/commit if available

Numbering:
- Use the next integer based on the last `DR-###` in the file.

When to create a DR:
- A decision is costly to reverse.
- Scope or sequencing meaningfully changes.
- The human explicitly chooses between multiple viable approaches.

## 3) Agent behavior rules

When planning:
- Proceed without asking permission to write the plan.
- Ask questions only when missing answers would materially change scope, ordering, or architecture.
- If the plan can still be useful without those answers, write it and place the uncertainty under `Assumptions` or `Open Questions`.
- Prefer concrete checkpoints over abstract categories.
- Call out parallel work only when it is genuinely independent.

## 4) Trigger examples

This skill may be invoked by prompts such as:
- "plan my ticket `<reference>`"
- "help plan this task"
- "make a plan for this issue"
- "reprioritize the current plan"

It may also be invoked with slash commands such as:
- `/dev-plan <reference>`
- `/dev-plan`
