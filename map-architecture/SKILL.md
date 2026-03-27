---
name: map-architecture
description: Maps a project or domain by tracing screens, interfaces, functions, callsites, branches, and data flow into Mermaid-based architecture notes. Use when the user asks for architecture analysis, domain mapping, dependency tracing, touchpoint discovery, or wants diagrams written under architecture/.
---

# Analyse Architecture
You are the project's architecture analyst.
Take a requested domain, or infer one from the task, then walk the codebase and map its touchpoints end to end.
Capture the result as durable Markdown and Mermaid diagrams under the closest git repo root in `architecture/`.

## When to run

Run this skill when:
- The user asks to map a domain, feature, module, flow, or screen architecture
- The user wants interfaces, functions, callsites, branches, or dependencies traced
- The user asks for Mermaid diagrams or architecture docs to be created
- The user asks for a high-level overview plus deeper drill-down diagrams

## Golden rules

1. Never write outside the closest git repo root.
2. Start with discovery before diagramming.
3. Trace concrete touchpoints: screens, entry points, interfaces, types, functions, callsites, conditionals, side effects, storage, network, and navigation.
4. Keep diagrams readable. Split large graphs by concern, but combine overlapping domains when that makes the model easier to follow.
5. Use sub-agents when the codebase or requested domain is large enough that parallel discovery will materially reduce time or blind spots. Never spawn more than 3 sub-agents.
6. Prefer evidence over interpretation. Mark inferred relationships explicitly.
7. Create an `overview` artifact whenever the codebase has multiple screens, routes, or top-level flows.
8. Diagrams support concise written findings; they are not the whole output.

## Output structure

Write to:
- `architecture/overview.md`
- `architecture/<domain>/README.md`
- `architecture/<domain>/*.md`

Use a stable filesystem-safe domain name such as `home`, `auth`, or `settings`.

## Workflow

### Step 1 - Find repo root and scope

- Identify the requested domain. If none is given, infer the most relevant bounded area from the user request and state that assumption.
- Determine whether the task needs one domain, multiple overlapping domains, or only a top-level overview.
- Decide whether the scope is small enough to map inline or large enough to justify delegation.

### Step 2 - Delegate discovery when warranted

Spawn sub-agents only when they help. Good triggers:
- The repo has several top-level apps, packages, or feature areas
- The requested domain spans multiple layers such as UI, state, backend integration, and persistence
- The user asked for multiple domains, or for both overview and deep drill-downs
- Early discovery shows too many files or call chains to inspect efficiently in one pass

Rules for delegation:
- Cap delegation at 3 sub-agents.
- Give each sub-agent a disjoint slice, for example one domain each, or UI vs data layer vs cross-cutting integrations.
- Keep the main agent responsible for the top-level map, synthesis, diagram structure, and final files.
- Do not wait idly. While sub-agents explore, continue with repo-level discovery, folder structure, and overview drafting.
- If the scope is small or tightly coupled enough that delegation would add coordination overhead, stay inline.

Ask sub-agents for concrete outputs:
- Entry points and key files
- Important call chains and dependencies
- Branching logic and side effects
- Open questions and inferred edges that still need verification

### Step 3 - Discover touchpoints

Trace the domain through the codebase. Include, where relevant:
- Screens, routes, views, and navigation entry points
- Public interfaces, protocols, services, stores, controllers, and models
- Functions, methods, event handlers, and async jobs
- Callsites and inbound callers
- Key conditionals, feature flags, guards, and branching paths
- Network requests, persistence, caching, and background work
- Cross-domain dependencies and shared abstractions

Use fast code search first, then open only the files needed to verify relationships.
When sub-agents are used, consolidate their findings into one verified model before diagramming. Resolve overlaps and contradictions explicitly rather than copying notes through unchanged.

### Step 4 - Group the architecture

Organise findings into the smallest useful set of diagrams, for example:
- Screen or route flow
- Dependency graph
- Data flow
- Decision or conditional flow
- External integration map

Combine diagrams when domains overlap heavily. Split diagrams when a single graph becomes hard to read.

### Step 5 - Write the domain docs

For each domain, create `architecture/<domain>/README.md` with:
- Scope
- Entry points
- Core components
- Key flows
- Conditionals and branching behaviour
- External dependencies
- Open questions or inferred edges

Add Mermaid diagrams in the same file or adjacent Markdown files when separate diagrams are clearer.

### Step 6 - Write the overview

Create `architecture/overview.md` when the project has more than one screen, route, or domain of interest.
Show how major screens or flows tie together and reference deeper docs such as `architecture/<domain>/README.md`.

### Step 7 - Confirm succinctly

Return:
- Domains analysed
- Files written
- Main architectural findings
- Any inferred or unresolved relationships

## Diagram guidance

- Use Mermaid.js flowcharts or sequence diagrams unless another Mermaid format is clearly better.
- Keep node labels short and descriptive.
- Prefer a few connected diagrams over one unreadable graph.
- If a relationship is inferred rather than directly observed, label it as inferred in the Markdown near the diagram.
