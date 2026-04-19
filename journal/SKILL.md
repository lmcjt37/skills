---
name: journal
description: Maintains a per-project engineering journal (journal.md), decision log, observation log, and session records across different agents and IDEs.
---

# Journal

## Overview

You are the project's *scribe*.

Your job: keep `journal.md` updated, **capture decisions**, and **store useful session records** without dumping noisy raw output.
The journal lives **per-project**, at the **closest git repo root**.

## When to run

Run this skill when:
- The user uses a slash command: `/journal <command>`
- The user says: **"update journal"** or **"journal log"**
- The user asks to capture observations or preserve session context for later handoff
- You detect a **high-signal decision** (commitment language or an implemented change that's difficult to revert)

## Golden rules

1. **Never write outside the repo root.** Determine the closest git root before writing anything.
2. **Keep `journal.md` readable.** Optimize for durable project memory, not exhaustive narration.
3. **Capture the session in the lightest useful form.** Prefer summaries and highlights over full raw transcripts unless verbatim history is genuinely needed.
4. **Decisions must be explicit.** Record the decision, why, alternatives, tradeoffs, and consequences.
5. **Observations are human-first.** Record observations when the human provides a meaningful comparison, metric, regression, or judgment.
6. **Redact secrets** automatically and leave a note indicating what was removed (e.g., “[REDACTED: API_TOKEN]”).
7. **Separate durable knowledge from session detail.** `journal.md` is the long-lived summary; session records capture what happened in a specific work period.
8. **Do not invent content to satisfy structure.** Omit empty sections rather than adding filler.

## Files you manage (all at repo root)

- `.journal/<branch>/journal.md` (curated narrative)
- `.journal/<branch>/decisions.md` (Decision Records, DR-###)
- `.journal/<branch>/observations.md` (Observation Records, OB-###)
- `.journal/<branch>/sessions/YYYY-MM-DD__<agent>.md` (session record with optional transcript excerpts and tool-output highlights)

## Workflow (do this every time)

### Step 1 — Find repo root + identity

- Repo root: `git rev-parse --show-toplevel`
- Branch: `git rev-parse --abbrev-ref HEAD` (sanitize `/` to `-`; if detached or unavailable, use `detached-head`)
- Date: local date (YYYY-MM-DD)
- Agent name: detect from system when easy; if unavailable use `[AGENT]`.

### Step 2 — Migrate legacy storage + ensure structure exists

Use `.journal/<branch>/` as the canonical storage location.

Before creating anything new, detect legacy storage at `journal/<branch>/`:
- If `journal/<branch>/` exists and `.journal/<branch>/` does not, move the full branch directory from `journal/<branch>/` to `.journal/<branch>/`.
- If both locations exist, merge carefully and preserve existing files in `.journal/<branch>/`. Only copy missing legacy content across; do not overwrite newer hidden-folder content blindly.
- If `journal/` becomes empty after migration, remove the empty legacy directories.

If missing after migration, create:
- `.journal/<branch>/journal.md` (with the required sections your outline specifies)
- `.journal/<branch>/decisions.md`
- `.journal/<branch>/observations.md`
- `.journal/<branch>/sessions/`

### Step 3 — Build the checkpoint payload

From the current conversation and recent work context, extract:

**A) Decisions**
Create a DR when:
- Commitment language appears (“we’ll do X”, “let’s go with Y because…”, “let’s skip Z because…”)
- Something is implemented that’s costly to revert (schema choice, architecture split, data model, CI approach, etc.)
- DR's aren't necessarily one per decision — if multiple related decisions are made in a session, they can be grouped into a single DR with sub-points.
- DR's don't need to be created after every implementation detail — focus on the high-signal decisions that shape the project.

**B) Observations**
Create an OB when:
- The human observes something about the implementation (comparisons, metrics, improvements, regressions, etc.), OR
- Slash command: `/observation <command>`
- These are reserved for human observations to keep them separate from agent-recorded decisions.
- If the human has not made a meaningful observation, do not create an OB.

**C) Session record**
Create a concise session record using the best available source:
- Chat history, IDE interaction history, issue comments, terminal activity, or a brief reconstruction from recent work
- If verbatim transcript data is unavailable, write a faithful summary instead of forcing transcript-like output
- If tool output is large, compress it into:
  - what command or tool ran / what changed
  - 5-15 key lines when useful
  - outcome + next step

**D) Journal narrative**
Update `journal.md` as a durable summary of the project state:
- Add a dated progress entry when something meaningful changed
- Refresh recurring lessons, risks, and key decisions when needed
- Prefer clear, direct writing; light personality is optional and should only improve readability

### Step 4 — Redact + write

Before writing, apply redaction rules from `references/guidelines.md`.
Write/update files in this order:
1) `.journal/<branch>/sessions/...`
2) `.journal/<branch>/decisions.md`
3) `.journal/<branch>/observations.md`
4) `.journal/<branch>/journal.md`

### Step 5 — Confirm succinctly

Return a short summary of what you wrote:
- session file name
- DR numbers added (if any)
- OB numbers added (if any)
- 1–2 sentence summary of Journal entry

## Output format (your response)

- **Checkpoint saved:** yes/no
- **Files updated:** list
- **Decisions added:** DR-### titles
- **Observations added:** OB-### titles
- **Notes:** anything you intentionally skipped/compressed/redacted

### references/
Documentation and reference material intended to be loaded into context to inform the agents process and thinking.

- Additional guidelines: `guidelines.md` - detailed workflow guides
