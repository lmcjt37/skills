# Agent Dev Plan — Reference

## 1) Required plan.md outline (create if missing)

plan.md MUST contain these sections:

1. JIRA reference.
2. Context for the feature. If none, ask for it.
3. Implementation steps broken down into consecutive bullet points (checkboxes).
4. Each bullet (checkbox) should have 1-4 sub-bullets of additional context.
5. There should always be a bullet point (checkbox) to verify changes (no regressions, profile, tests pass).
6. List any DR-XXX references in a comma separated list at the bottom of the relevant implementation section.
7. At the bottom of the plan there should always be a final checkbox to re-audit for improvements.

Style:
- Succinct but clear detail
- Human readable

Template for each section:
- x [ ] **Impl X - Section title**: 1–2 sentence description of the implementation step.
    - x.1 [ ] Key note, dependency, or risk #1
    - x.2 [ ] Key note, dependency, or risk #2
    - x.3 [ ] Key note, dependency, or risk #3
    - DR references: DR-XXX

## 2) Decision Record format (plan/<branch>/decisions.md) (create if missing)

Each decision uses this template:

## DR-### — <Decision Title>
- Date: YYYY-MM-DD
- Branch: <branch>
- Context: what problem were we solving?
- Decision: what did we choose?
- Why: constraints + reasoning
- Alternatives considered: A/B/C + why not
- Tradeoffs: gains vs costs
- Metrics: Any measurements given (omit key if none)
- Consequences: what this enables / complicates
- Links: PR/issue/commit if available

Numbering:
- Use the next integer based on the last DR-### in the file.

## 3) Other triggers

This skill is designed to be invoked by a wrapper command like:
- "plan my ticket `<JIRA-REFERENCE>`"
- "Can you help plan my ticket `<JIRA-REFERENCE>`"

OR

Invoked with slash commands like:
- `/plan <JIRA-REFERENCE>`