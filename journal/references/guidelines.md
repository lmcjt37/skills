# Agent Dev Journal — Reference

## 1) Required `journal.md` outline

`journal.md` should be a durable project memory, not a transcript dump.

Include these sections when creating or substantially rewriting the journal:

1. `Project Snapshot`
2. `Recent Progress`
3. `Key Decisions`
4. `Recurring Lessons`
5. `Open Risks / Follow-ups`

Guidance:
- Keep the writing clear, direct, and useful to humans first.
- Light personality is optional. Do not force humor, analogies, or “war stories”.
- Update durable sections only when the session meaningfully changes them.
- Prefer one dated progress entry per meaningful session rather than scattering notes across many sections.
- Omit empty sections rather than filling them with generic text.

Recommended template:

```md
# Engineering Journal

## Project Snapshot
- Project / workstream: <what this effort is about>
- Current focus: <what is being worked on now>
- Constraints: <important product, technical, or process constraints>

## Recent Progress
### YYYY-MM-DD
- What changed
- Why it mattered
- What remains or what is next

## Key Decisions
- DR-001 — <title>

## Recurring Lessons
- <pitfall, pattern, or practice worth remembering>

## Open Risks / Follow-ups
- <important unresolved items>
```

## 2) Decision Record format (`journal/<branch>/decisions.md`)

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

## 3) Observation Record format (`journal/<branch>/observations.md`)

Observations are for human-reported judgments, metrics, comparisons, regressions, or improvements.
Do not create one unless the human actually supplied a meaningful observation.

Each observation should use this template:

## OB-### — <Observation Title>
- Date: YYYY-MM-DD
- Branch: <branch>
- Context: what problem were we solving?
- Observation: what did we observe?
- Why: any reasoning given
- Metrics: measurements given, if any
- Links: PR/issue/doc/commit if available

Numbering:
- Use the next integer based on the last `OB-###` in the file.

## 4) Session record format (`journal/<branch>/sessions/*`)

Filename:
`YYYY-MM-DD__<agent>.md`

Agent fallback token:
`[AGENT]`

The session record should capture the work in the lightest useful form. It does not need to be a verbatim transcript.

Recommended template:

```md
# Session Record — YYYY-MM-DD — Branch: <branch> — Agent: <agent>

## Summary
- 3-7 bullets on what happened

## Changes Made
- Files, systems, or behaviors touched
- Important commands, tools, or artifacts when relevant

## Decisions Captured
- DR-### — <title>

## Human Observations
- OB-### — <title>

## Open Questions / Next Steps
- [ ] ...
```

If raw transcript data is available and useful:
- Include only the relevant excerpts.
- Compress large tool output into key lines and outcomes.
- Prefer summarized commands/results over dumping full logs.

## 5) Redaction rules

Goal: keep useful history while preventing secret leakage.

Replace sensitive content with bracketed notes such as:
- `[REDACTED: API_TOKEN]`
- `[REDACTED: PASSWORD]`
- `[REDACTED: PRIVATE_KEY]`
- `[REDACTED: AUTH_HEADER]`
- `[REDACTED: SESSION_COOKIE]`

At minimum redact:
- Credentials in env vars such as `*_KEY`, `*_TOKEN`, `*_SECRET`, `PASSWORD`, `AUTH`, `BEARER`
- Private keys or certificate blocks such as `BEGIN PRIVATE KEY`
- Authorization headers
- Session cookies
- Long opaque tokens when preceded by auth-like labels

Optional redaction if appropriate:
- Emails
- Internal hostnames
- Customer names

Always leave the redaction note indicating the type of data removed.

## 6) High-signal filter

Only create or substantially update journal artifacts when one of these is true:
- An irreversible or expensive-to-reverse choice was made
- A non-trivial bug or gotcha was found and fixed
- A new pattern or practice was learned that will recur
- A human observation is worth preserving for later comparison
- There is important context that a later handoff would otherwise miss

Minor work does not always require a new session record or journal update.

## 7) Agent behavior rules

When journaling:
- Prefer accurate summaries over colorful narration.
- Write for a future human teammate first, then for future agents.
- If raw interaction history is incomplete, write a faithful reconstruction instead of pretending you have a full transcript.
- Keep durable project context in `journal.md`; keep session-specific details in `sessions/`.
- Do not create empty DR or OB sections just to satisfy the template.

## 8) Trigger examples

This skill may be invoked by prompts such as:
- "journal log"
- "update the journal"
- "capture this session"
- "record this observation"

It may also be invoked with slash commands such as:
- `/journal <command>`
- `/observation <comment>`
