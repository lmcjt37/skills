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
9. Default to behavioral depth, not just structural breadth. A useful map explains what happens during lifecycle, success, failure, and state transitions.
10. Treat screens and flows as first-class units. For UI areas, map what the user sees, what triggers work, what functions run, and what state is rendered.

## Output structure

Write to:
- `architecture/overview.md`
- `architecture/<domain>/README.md`
- `architecture/<domain>/screens/<screen>.md`
- `architecture/<domain>/flows/<flow>.md`
- `architecture/<domain>/*.md`
- `architecture/cosmograph/points.json`
- `architecture/cosmograph/links.json`
- `architecture/cosmograph/config.json`
- `architecture/cosmograph/points.indexed.json`
- `architecture/cosmograph/links.indexed.json`
- `architecture/cosmograph/config.indexed.json`
- `architecture/cosmograph/layout.json`
- `architecture/cosmograph/config.layout.json`

Use a stable filesystem-safe domain name such as `home`, `auth`, or `settings`.
Use stable filesystem-safe screen and flow names such as `checkout-summary`, `profile-edit`, or `pull-to-refresh`.

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

For each major screen, flow, or entry point, also extract:
- Lifecycle triggers such as initial load, appear/mount, focus, refresh, retry, submit, background resume, and teardown
- The ordered function or method chain invoked by each trigger
- Success paths, failure paths, empty states, loading states, and disabled states
- State producers and consumers: view model state, store slices, derived values, selectors, bindings, and props
- Error handling behavior: where errors are caught, transformed, ignored, surfaced to UI, retried, or logged
- Helpers, utilities, formatters, adapters, and mappers used by the area, plus what each helper is responsible for
- Tight coupling points: files that change together, files that know too much about each other, and cross-layer shortcuts
- Architectural patterns in use, such as MVVM, Redux-style store, coordinator, service layer, repository, observer, dependency injection, or ad hoc patterns
- Reasonable inferred intent behind conditional logic or structure, for example performance tradeoffs, backward compatibility, staged rollout, defensive validation, or UI consistency

Do not stop at naming components. Follow execution.
If a screen or flow exists, identify what the user action or lifecycle event is, what code path it enters, what state changes occur, and what UI can be emitted as a result.

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
For non-trivial domains, split the written analysis into focused screen-level and flow-level artifacts instead of collapsing everything into one long README.

### Step 5 - Write the domain docs

For each domain, create `architecture/<domain>/README.md` with:
- Scope
- Entry points
- Screen map
- Flow map
- Inferred architecture patterns
- Core components
- Coupling and change-risk hotspots
- External dependencies
- Links to screen and flow deep dives
- Open questions or inferred edges

Add Mermaid diagrams in the same file or adjacent Markdown files when separate diagrams are clearer.
Prefer multiple focused files over one shallow summary.

For non-trivial domains, also create:
- `architecture/<domain>/screens/<screen>.md` for each important screen, route, or view container
- `architecture/<domain>/flows/<flow>.md` for each important lifecycle, user journey, async process, submission path, sync path, or error-recovery path

Use the domain README as the index and synthesis layer, not the place to dump every detail.

Use this structure by default for non-trivial domains:

```md
# <Domain>
## Scope
## Entry Points
## Screen Map
## Flow Map
## Architecture Pattern
## Lifecycle Flows
## Coupling
## External Dependencies
## Screen Deep Dives
## Flow Deep Dives
## Open Questions
```

Within those sections:
- Name concrete files and symbols, not just layers
- Show ordered call sequences where possible
- Separate observed facts from inferred intent
- Call out missing or inconsistent state handling if discovered
- When a view state or error path is implied but not directly rendered in code, mark it as inferred

Use this structure by default for `screens/<screen>.md`:

```md
# <Screen>
## Purpose
## Entry Conditions
## Rendered States
## Lifecycle Triggers
## Function Call Chains
## State Sources and Sinks
## Success and Error Handling
## Helpers
## Coupled Files
## Conditionals and Inferred Decisions
## Open Questions
```

Use this structure by default for `flows/<flow>.md`:

```md
# <Flow>
## Purpose
## Start Trigger
## Participating Files
## Ordered Execution Path
## State Transitions
## Success Outcome
## Error Outcome
## Retry or Recovery Behavior
## Helpers and Shared Logic
## Coupling and Risk
## Inferred Design Rationale
## Open Questions
```

Screen docs should answer:
- What the user sees
- What starts work
- Which functions run in order
- Which states can render
- How success, empty, loading, and error outcomes differ

Flow docs should answer:
- What kicks the flow off
- Which files and symbols participate
- How data and control move step by step
- Where branching, retries, or failure handling happen
- Why the implementation may have been structured this way

### Step 6 - Write the overview

Create `architecture/overview.md` when the project has more than one screen, route, or domain of interest.
Show how major screens or flows tie together and reference deeper docs such as `architecture/<domain>/README.md`.

### Step 7 - Generate Cosmograph exports for full-codebase overviews

When you map the whole codebase, also generate exportable graph data for Cosmograph in `architecture/cosmograph/`.
Do this for full-codebase overviews, not for every small single-domain mapping unless the user asks for it.
Generate both export modes unless the user explicitly asks for only one:
- Raw JSON plus Data Kit mapping config
- Fully indexed v2-ready JSON plus direct render config

Use one combined graph dataset for each export mode:
- One `points` dataset
- One `links` dataset
- Do not split the graph into separate overview and behavior files
- Instead, encode both overview structure and deeper behavioral detail inside the same dataset using metadata fields and pruning rules

Write:
- `architecture/cosmograph/points.json`
- `architecture/cosmograph/links.json`
- `architecture/cosmograph/config.json`
- `architecture/cosmograph/points.indexed.json`
- `architecture/cosmograph/links.indexed.json`
- `architecture/cosmograph/config.indexed.json`
- `architecture/cosmograph/layout.json`
- `architecture/cosmograph/config.layout.json`

Use Cosmograph-compatible raw JSON tables:
- The official docs say `points` and `links` can be provided as `Record<string, unknown>[]`, including JSON arrays of objects.
- The minimal config should map `pointIdBy` for points plus `linkSourceBy` and `linkTargetsBy` for links.
- The v2 migration docs say direct rendering requires `pointIndexBy` on points and `linkSourceIndexBy` plus `linkTargetIndexBy` on links.

Default file contents:

`points.json`
- JSON array of point objects
- Each point must include `id`
- Prefer also including `label`, `kind`, `domain`, `path`, `notes`, `layer`, `cluster`, `subcluster`, `graphLevel`, `importance`, and `sizeWeight` when available

`links.json`
- JSON array of link objects
- Each link must include `source` and `target`
- Prefer also including `relationship`, `label`, `inferred`, `evidence`, `flow`, `graphLevel`, and `strength` when available

`config.json`
- JSON object shaped for Cosmograph data preparation
- Use:
```json
{
  "points": {
    "pointIdBy": "id"
  },
  "links": {
    "linkSourceBy": "source",
    "linkTargetsBy": ["target"]
  }
}
```

`points.indexed.json`
- JSON array of point objects for direct Cosmograph v2 rendering
- Each point must include:
  - `id`
  - `index`
- `index` must be a unique zero-based ordinal integer aligned with the full points array
- Prefer also including `label`, `kind`, `domain`, `path`, `notes`, `layer`, `cluster`, `subcluster`, `graphLevel`, `importance`, and `sizeWeight`

`links.indexed.json`
- JSON array of link objects for direct Cosmograph v2 rendering
- Each link must include:
  - `source`
  - `target`
  - `sourceIndex`
  - `targetIndex`
- `sourceIndex` and `targetIndex` must match the `index` values of the referenced points
- Cosmograph v2 expects single source-target pairs, so flatten any multi-target relationship into separate link rows
- Prefer also including `relationship`, `label`, `inferred`, `evidence`, `flow`, `graphLevel`, and `strength`

`config.indexed.json`
- JSON object for direct Cosmograph rendering against indexed JSON
- Use:
```json
{
  "pointIdBy": "id",
  "pointIndexBy": "index",
  "linkSourceBy": "source",
  "linkSourceIndexBy": "sourceIndex",
  "linkTargetBy": "target",
  "linkTargetIndexBy": "targetIndex"
}
```

`layout.json`
- JSON object containing deterministic cluster placement metadata
- Use `clusterPositionsMap` keyed by major domain cluster id
- Use stable coordinates so major domains keep the same relative placement across runs when the same codebase shape is mapped
- Prefer a readable spread such as left-to-right lanes, grid sectors, or hub-and-spoke placement depending on the repo shape
- Include optional notes about the placement strategy when useful

Example:
```json
{
  "clusterPositionsMap": {
    "auth": [-1200, 200],
    "home": [-200, 200],
    "payments": [900, -100],
    "settings": [900, 700]
  },
  "strategy": "major-domains-grid"
}
```

`config.layout.json`
- JSON object for reusable Cosmograph layout and simulation tuning
- Use deterministic major-domain clustering plus lighter intra-domain separation
- Include:
  - `pointClusterBy`: the point column representing the major domain, usually `domain`
  - `clusterPositionsMap`: copied from `layout.json`
  - `simulationClusterStrength`: enough to preserve deterministic domain grouping without overcompressing points
  - `simulationRepulsion`: increased moderately to separate points within each domain cluster
  - `simulationLinkDistance`: increased moderately so local flow/state/helper nodes are not packed too tightly
- Optionally include `pointClusterStrengthBy` if the graph intentionally varies pull strength by point type or importance

Example:
```json
{
  "pointClusterBy": "domain",
  "clusterPositionsMap": {
    "auth": [-1200, 200],
    "home": [-200, 200],
    "payments": [900, -100],
    "settings": [900, 700]
  },
  "simulationClusterStrength": 0.65,
  "simulationRepulsion": 0.25,
  "simulationLinkDistance": 3
}
```

Model the codebase as one readable combined architecture graph with enough depth to reflect both the overview and the deeper screen and flow analysis:
- Points should represent the important architectural and behavioral entities in the map, not just top-level modules.
- Include points where relevant for:
  - apps, packages, domains, routes, screens, and view containers
  - flows and lifecycle triggers
  - rendered states such as loading, success, empty, disabled, and error
  - modules, services, stores, controllers, repositories, APIs, databases, queues, and external systems
  - helpers, mappers, adapters, formatters, validators, and error handlers
- Use `kind` to distinguish point types such as `domain`, `screen`, `flow`, `trigger`, `state`, `service`, `store`, `helper`, `api`, `database`, `external`, or `error_handler`.
- Use `layer` when useful to distinguish UI, orchestration, domain, data, platform, or external concerns.
- Use `cluster` for the major deterministic placement group, usually the same as `domain`.
- Use `subcluster` for local grouping within the domain, for example `ui`, `flow`, `state`, `service`, `helper`, or `external`.
- Use `graphLevel` to distinguish nodes that are part of the top-level overview from nodes that add deeper behavioral detail. Prefer values such as `overview` and `behavior`.
- Use `importance` to distinguish `high`, `medium`, and `low` signal nodes.
- Use `sizeWeight` as a numeric hint for later rendering or filtering.
- Prefer fewer high-signal points over a noisy file-by-file dump unless the user explicitly wants file-level granularity.

Links should represent concrete structural and behavioral relationships such as:
- `owns`
- `contains`
- `navigates_to`
- `triggers`
- `calls`
- `depends_on`
- `reads`
- `writes`
- `emits`
- `subscribes_to`
- `renders`
- `transitions_to`
- `handles_error_with`
- `uses_helper`
- `guards`
- `retries`
- `implements`
- `coupled_to`

Use link metadata deliberately:
- Use `graphLevel` to distinguish overview edges from deeper behavioral edges.
- Use `strength` as a numeric hint for prominent versus secondary relationships.
- Use `flow` when the edge belongs to a named lifecycle or user journey.

When deeper mapping is available:
- Represent major screens as points.
- Represent important flows as points.
- Represent major rendered states as points when they clarify behavior.
- Represent helpers and error-handling components as points when they materially shape control flow.
- Add links that show which trigger starts a flow, which flow calls which functions or services, which states can be rendered, where errors move, and which files or components are tightly coupled.
- Use `notes` on points for concise inferred rationale or implementation context when it helps future reuse.

Cosmograph exports should stay aligned with the deeper docs:
- If `screens/<screen>.md` exists, the graph should usually include a corresponding `screen` point.
- If `flows/<flow>.md` exists, the graph should usually include a corresponding `flow` point.
- If the docs identify distinct loading, success, empty, or error states, include corresponding `state` points when they materially help the graph explain behavior.
- If coupling or helper usage is called out in the docs, encode the important relationships in the graph instead of leaving them only in prose.
- Major domains should usually be clusterable by a shared `domain` field so deterministic domain placement is possible.

The combined graph should remain readable:
- `overview` nodes and edges should form the backbone of the graph.
- `behavior` nodes and edges should enrich the local picture, not drown the backbone.
- Behavioral nodes should usually attach to a parent screen, flow, service, or domain instead of floating as isolated graph noise.
- Do not emit low-value helper, state, or trigger nodes unless they materially clarify lifecycle, rendering, error handling, or coupling.
- Prefer one high-signal node with good metadata over several near-duplicate nodes.

Do not try to encode every function as a point by default.
Use flow, state, helper, and coupling nodes selectively so the graph becomes more explanatory, not more cluttered.
Keep ids stable and filesystem-safe where possible.
Mark uncertain relationships with `inferred: true` and include short evidence or rationale.

For layout and spacing:
- Use `pointClusterBy` with the major domain field, usually `domain`, so the graph can cluster by major area.
- Emit `clusterPositionsMap` so major domains have deterministic positions across runs.
- Keep `overview` nodes visually central within each domain cluster and allow `behavior` nodes to sit slightly more peripherally through moderate repulsion and link distance.
- Use `simulationRepulsion` and `simulationLinkDistance` to slightly separate nodes within each domain cluster rather than relying only on cluster placement.
- Keep force tuning moderate. The goal is readable domain interiors, not exploding related nodes so far apart that flows become hard to follow.
- Prefer consistent placement strategies across iterations of the same repo, for example auth on the left, primary product flows in the center, platform or settings to the right, external systems at the periphery.
- If some points should stay closer to the cluster center, optionally support `pointClusterStrengthBy` with lower values for peripheral helper or state nodes and higher values for core screen or flow nodes.
- Do not provide manual coordinates for every point unless the user explicitly asks for a hand-authored layout. Keep determinism at the domain-cluster level and let the simulation handle local arrangement.

The goal is not just validity but reuse:
- The JSON should be directly reusable later in a Cosmograph workflow.
- Keep the schema consistent within one export.
- Ensure every `source` and `target` refers to an existing point `id`.
- Ensure every `sourceIndex` and `targetIndex` refers to an existing point `index`.
- Keep the raw and indexed exports semantically aligned so they describe the same graph at different preparation levels.
- Keep `layout.json` and `config.layout.json` aligned with the `domain` values actually present on points.

### Step 8 - Confirm succinctly

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
- For UI domains, prefer at least these diagrams when evidence supports them:
  - Screen navigation or route map
  - Lifecycle sequence from user or framework trigger to rendered state
  - State transition or decision flow for loading, success, empty, and error outcomes
  - Dependency or coupling graph for the files that jointly implement the area
