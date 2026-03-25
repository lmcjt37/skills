# Agent Dev Journal — Reference

## 1) Required journal.md outline (create if missing)

Journal.md MUST contain these sections:

1. The Big Picture
2. Architecture Deep Dive
3. The Codebase Map
4. Tech Stack & Why
5. The Journey
6. Engineer's Wisdom
7. If I Were Starting Over...

Style:
- engaging, educational, medium-fun
- analogies encouraged
- include “war stories”
- always capture the WHY, not just the WHAT

## 2) Decision Record format (journal/<branch>/decisions.md)

Each decision uses this template:

## DR-### — <Decision Title>
- Date: YYYY-MM-DD
- Branch: <branch>
- Context: what problem were we solving?
- Decision: what did we choose?
- Why: constraints + reasoning
- Alternatives considered: A/B/C + why not
- Tradeoffs: gains vs costs
- Consequences: what this enables / complicates
- Links: PR/issue/commit if available

Numbering:
- Use the next integer based on the last DR-### in the file.

## 3) Observation Record format (journal/<branch>/observations.md)

Each observation uses this template:

## OB-### — <Observation Title>
- Date: YYYY-MM-DD
- Branch: <branch>
- Context: what problem were we solving?
- Observation: what did we observe?
- Why: any reasoning
- Metrics: what measurements were gievn if any (omit this key if none)
- Links: PR/issue/commit if available

Numbering:
- Use the next integer based on the last OB-### in the file.

## 4) Transcript format (journal/<branch>/transcripts/*)

Filename:
YYYY-MM-DD__<agent>.md
- Agent fallback token: [AGENT]

Contents template:

# Session Transcript — YYYY-MM-DD — Branch: <branch> — Agent: <agent>
## Summary
- Bullet list (3–7) of what happened

## Full Conversation (with tool-output compression)
> User: ...
> Agent: ...

### Tool Output (highlighted)
If tool output is large, store only:
- Command / tool name
- 5–15 key lines
- Outcome
- Link to artifact (if any)

## Decisions Extracted
- DR-### — Title
- DR-### — Title

## TODO / Next actions
- [ ] ...

## 5) Redaction rules (must apply before writing)

Goal: keep useful history while preventing secret leakage.

Replace with bracketed notes like:
- [REDACTED: API_TOKEN]
- [REDACTED: PASSWORD]
- [REDACTED: PRIVATE_KEY]
- [REDACTED: AUTH_HEADER]
- [REDACTED: SESSION_COOKIE]

At minimum redact:
- Anything that looks like credentials in env vars: *_KEY, *_TOKEN, *_SECRET, PASSWORD, AUTH, BEARER
- Private keys/certs blocks (BEGIN PRIVATE KEY)
- Authorization headers
- Session cookies
- Long opaque tokens (e.g., 20+ chars of base64/hex-looking strings) when preceded by auth-ish labels

Optional redaction (enable if requested later):
- Emails
- Internal hostnames
- Customer names

Always leave the redaction note indicating type.

## 6) High-signal filter (avoid journaling junk)

Only create DRs/entries if one of:
- Irreversible/expensive-to-revert choice was made
- Non-trivial bug/gotcha + fix
- New pattern/practice learned that will recur
- There is clear “why” future-you will forget

## 7) journal.md entry constraints

- 400–500 words max per entry
- Medium fun: natural analogies, occasional humor
- Prefer clarity over cleverness
- Include: What happened, Why it mattered, What we learned, Pitfalls

## 8) Other triggers

This skill is designed to be invoked by a wrapper command like:
- "journal log"
- "journal"

OR

Invoked with slash commands like:
- `/journal <command>`