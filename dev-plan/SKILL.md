---
name: dev-plan
description: Maintains a per-project engineering plan (plan.md) plus a structured decision log, adapting to tickets, docs, or local tasks across different agents and IDEs.
---

# Dev Plan

## Overview

You are the project's *Architect*.

Your job: maintain a clear implementation plan in `plan.md` and capture important choices in `decisions.md`.
The plan lives **per-project**, at the **closest git repo root**, and should keep work structured across sessions for both humans and agents.

## When to run

Run this skill when:
- The user uses a slash command such as `/dev-plan <task reference>`
- The user says: **"plan my ticket <reference>"**, **"help plan this task"**, or **"make a plan"**
- The user asks to reprioritize, skip, or refine parts of an existing plan
- You detect a **high-signal planning decision** (scope, order, architecture, or approach commitments)

## Golden rules

1. **Never write outside the repo root.** Determine the closest git root before writing anything.
2. **Keep `plan.md` concise and actionable.** Optimize for fast human scanability and clean agent handoff.
3. **Plan in checkpoint-sized chunks.** Each chunk should be small enough to implement and commit safely.
4. **Decisions must be explicit.** Record the decision, rationale, alternatives, tradeoffs, and consequences.
5. **Stay adaptable.** If the user asks to skip or reprioritize, update the plan instead of restarting from scratch.
6. **Prefer the best available context source.** Use the user prompt, linked ticket/doc, repo docs, codebase context, and optional integrations in that order.
7. **Separate facts from assumptions.** If context is incomplete, write assumptions explicitly instead of presenting guesses as settled facts.
8. **Only ask questions when the answers would materially change the plan.** Otherwise proceed and record open questions.

## Files you manage (all at repo root)

- `.plan/<branch>/plan.md` (curated implementation plan)
- `.plan/<branch>/decisions.md` (Decision Records, DR-###)

## Workflow (do this every time)

### Step 1 — Find repo root + identity

- Repo root: `git rev-parse --show-toplevel`
- Branch: `git rev-parse --abbrev-ref HEAD` (sanitize `/` to `-`; if detached or unavailable, use `detached-head`)
- Task reference: parse from user input, linked issue, document, or branch name when provided
- Agent name: optional metadata; detect from system when easy, otherwise omit it

### Step 2 — Migrate legacy storage + ensure structure exists

Use `.plan/<branch>/` as the canonical storage location.

Before creating anything new, detect legacy storage at `plan/<branch>/`:
- If `plan/<branch>/` exists and `.plan/<branch>/` does not, move the full branch directory from `plan/<branch>/` to `.plan/<branch>/`.
- If both locations exist, merge carefully and preserve existing files in `.plan/<branch>/`. Only copy missing legacy content across; do not overwrite newer hidden-folder content blindly.
- If `plan/` becomes empty after migration, remove the empty legacy directories.

If missing after migration, create:
- `.plan/<branch>/plan.md`
- `.plan/<branch>/decisions.md`

### Step 3 — Gather context

- Gather the best available context from the prompt, issue tracker, design docs, repo docs, and codebase.
- Extract the goal, constraints, acceptance criteria, relevant existing patterns, risks, and verification expectations.
- If the context source is unavailable, continue with what you have and note the missing source in the plan or response.

### Step 4 — Decide whether to ask questions

- If missing information would materially change scope, sequencing, or architecture, ask focused questions before writing or heavily revising the plan.
- If the gaps are non-blocking, proceed and record them under `Assumptions` or `Open Questions`.
- Do not ask for permission to do the planning work. Provide a concrete proposal the human can react to.

### Step 5 — Build the planning payload

From the available context, extract:

**A) Implementation sections**
Create a practical plan broken into small deliverable chunks:
- Use 3-6 meaningful checkpoints by default; fewer for small tasks, more only when the work clearly warrants it.
- Use a single checkbox per checkpoint with optional supporting sub-bullets for dependencies, risks, or notes.
- Prefer simple sequencing over forced nesting. Only call out parallel work when it is genuinely safe and useful.
- Ensure the plan supports smooth handoff between sessions and between human and agent collaborators.
- Read `references/guidelines.md` for the required plan structure and templates.

**B) Decision records**
Create a DR when:
- Commitment language appears ("we'll do X", "let's skip Y", "reprioritise Z"), OR
- A plan choice is expensive to reverse (architecture, data model, rollout approach, integration strategy).
- Read `references/guidelines.md` for the DR template and numbering rules.

**C) Reprioritisation updates**
When the user asks to adjust scope/order:
- Verify the requested change if it is ambiguous or risky.
- Preserve existing useful plan content.
- Reorder, split, or remove bullet points as requested.
- Add/update DR entries when rationale materially changes.

### Step 6 — Write updates

Write/update files in this order:
1) `.plan/<branch>/plan.md`
2) `.plan/<branch>/decisions.md`

### Step 7 — Confirm succinctly

Return a short summary of what you wrote:
- Artifacts modified with paths.
- DR numbers added (if any)
- 1–2 sentence summary of what changed in the plan.
- Mark off the checkboxes if/when implementation is verified.

## Output format (your response)

- **Checkpoint saved:** yes/no
- **Files updated:** list
- **Decisions added:** DR-### titles
- **Plan updates:** summary of added/updated/reordered sections
- **Notes:** anything intentionally deferred (for example, missing ticket detail or unavailable MCP context)

### references/
Documentation and reference material intended to be loaded into context to inform the agents process and thinking.

- Additional guidelines: `guidelines.md` - detailed workflow guides
