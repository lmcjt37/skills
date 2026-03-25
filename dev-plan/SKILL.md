---
name: dev-plan
description: Maintains a per-project engineering plan (plan.md) plus a structured decision log.
---

# Dev Plan

## Overview

You are the project's *Architect*.

Your job: maintain a clear implementation plan in `plan.md` and capture important choices in `decisions.md`.
The plan lives **per-project**, at the **closest git repo root**, and should keep ticket work structured across sessions.

## Prerequisites

- Atlassian MCP -> Required for extracting context.

## When to run

Run this skill when:
- The user uses a slash command: `/dev-plan <JIRA-REFERENCE>`
- The user says: **"plan my ticket <JIRA-REFERENCE>"** or **"help plan my ticket <JIRA-REFERENCE>"**
- The user asks to reprioritize, skip, or refine parts of an existing plan
- You detect a **high-signal planning decision** (scope, order, architecture, or approach commitments)

## Golden rules

1. **Never write outside the repo root.** Determine the closest git root before writing anything.
2. **Keep `plan.md` concise and actionable.** Use bullet points for implementation sections and sub-bullets for context.
3. **Plan in checkpoint-sized chunks.** Each chunk should be small enough to implement and commit safely.
4. **Decisions must be explicit.** Record the decision, rationale, alternatives, tradeoffs, and consequences.
5. **Stay adaptable.** If the user asks to skip or reprioritize, update the plan instead of restarting from scratch.
6. **Use Atlassian MCP for ticket context** when available; if unavailable, notify the user that they should connect it.

## Files you manage (all at repo root)

- `plan/<branch>/plan.md` (curated implementation plan)
- `plan/<branch>/decisions.md` (Decision Records, DR-###)

## Workflow (do this every time)

### Step 1 — Find repo root + identity

- Repo root: `git rev-parse --show-toplevel`
- Branch: `git rev-parse --abbrev-ref HEAD` (sanitize `/` to `-`)
- Ticket reference: parse from user input (`<JIRA-REFERENCE>`) when provided
- Agent name: detect from system; if unavailable use `[AGENT]`

### Step 2 — Ensure structure exists

If missing, create:
- `plan/<branch>/plan.md`
- `plan/<branch>/decisions.md`

### Step 3 — Plan proposal

- Analyse the context of the ticket (prefer Atlassian MCP) (description, acceptance criteria, technical considerations, test considerations, etc).
- Ask any necessary questions needed for extra context or clarity ("Are there similar patterns to follow?", "What about accessibility?", "Are there designs we can link to?", etc).
- Propose a high level breakdown of the implementation steps you think we should take, where appropriate also state which steps can be worked on in parallel.
- **Important**, ask for permission to proceed in planning and writing the implementation steps.

### Step 4 — Build the planning payload

From the proposal and current codebase context, extract:

**A) Implementation sections**
Create a practical plan broken into small deliverable chunks:
- One section per meaningful implementation checkpoint.
- Each section should have a main bullet point (checkbox) with consecutive numbering for marking off later.
- Each section should also have 1-4 sub-bullets (checkbox) for key notes, dependencies, and risks.
- Sections should support smooth handoff between sessions.
- refer to guidelines.md for more detailed instructions on how to structure the plan sections and sub-bullets.

**B) Decision records**
Create a DR when:
- Commitment language appears ("we'll do X", "let's skip Y", "reprioritise Z"), OR
- A plan choice is expensive to reverse (architecture, data model, rollout approach, integration strategy).
- refer to guidelines.md for more detailed instructions on how to structure the plan sections and sub-bullets.

**C) Reprioritisation updates**
When the user asks to adjust scope/order:
- **Always** verify the decision before implementation.
- Preserve existing useful plan content.
- Reorder, split, or remove bullet points as requested.
- Add/update DR entries when rationale materially changes.

### Step 5 — Write updates

Write/update files in this order:
1) `plan/<branch>/plan.md`
2) `plan/<branch>/decisions.md`

### Step 6 — Confirm succinctly

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
