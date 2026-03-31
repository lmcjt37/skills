---
name: cosmograph
description: Walks a codebase and produces a render-ready Cosmograph architecture dataset by extracting meaningful points, typed links, config, and optional layout under architecture/output/.
---

# Cosmograph Architecture Mapping

You are the project's architecture cartographer for Cosmograph.
Your job is to walk the code carefully, identify meaningful architectural entities and relationships, and emit a graph dataset that renders clearly and answers useful engineering questions.

The goal is not to graph every symbol in the codebase.
The goal is to represent the application's architecture at the right level of abstraction for navigation, reasoning, and change impact analysis.

## When to run

Run this skill when:
- The user asks for a Cosmograph of an application, feature, or architecture
- The user wants `points.json`, `links.json`, `config.json`, or `layout.json`
- The user wants a codebase explored and represented as a graph
- The user wants touchpoints, dependencies, navigation, or data flow mapped into Cosmograph

## Golden rules

1. Define `SCAN_ROOT` as the current working directory at skill start.
2. `SCAN_ROOT` is the explicit and authoritative starting scope. Do not silently widen from the current working directory to a repo root or sibling subtree.
3. Never read, trace, or classify files outside the `SCAN_ROOT` subtree unless a direct dependency or transient dependency from an in-scope touchpoint requires it to complete a truthful architectural path.
4. Treat the closest git repo root as the write root for all generated output, not as the scan scope.
5. Never write outside the closest git repo root.
6. Start with discovery. Do not emit graph data until you understand the architecture well enough to defend the node and link choices.
7. Stay architecture-pattern agnostic. Detect the architecture that exists instead of forcing the codebase into a preconceived pattern.
8. Represent meaningful architectural entities, not every implementation detail.
9. Bias toward denser architectural coverage once an entity or relationship is meaningful. Prefer more truthful detail over an overly sparse graph.
10. Every link must have semantic meaning. Avoid generic unlabeled edges.
11. Prefer evidence over interpretation. Mark inferred edges or classifications explicitly.
12. Keep the graph renderable. If a choice would create noise without insight, collapse or omit it.
13. Use stable IDs and stable indices so repeated runs produce comparable output.
14. If the codebase already has useful architecture docs under `architecture/`, use them as supporting context, but verify against code before emitting the graph.
15. If layout is obvious from the graph shape, omit `layout.json`. Only create it when it materially improves readability.
16. The output should be useful both for visual rendering and for downstream filtering, grouping, and drill-down behavior.
17. If it materially improves coverage and the environment supports it, you may spawn up to 3 sub-agents to crawl independent areas of the codebase in parallel. Any sub-agent must inherit the same `SCAN_ROOT` restriction.
18. The minimum acceptable coverage is full route-to-boundary architecture coverage for the scanned area, including at least one point for every route, container, screen, and meaningful child view in scope.
19. If the codebase uses builder, factory, coordinator, assembler, or view-model patterns, those orchestration touchpoints are first-class candidates and should usually be represented as points.
20. A domain is not complete until you have explicitly audited steady-state flows plus lifecycle, loading, empty, error, modal, and cross-domain navigation paths where they exist.
21. When you believe the map is complete, perform a deliberate re-audit of the domain to look for missing routes, branches, state transitions, and indirect touchpoints before finalizing output.
22. Use asymmetric granularity. Keep core app composition and user flows highly granular, but collapse dependency internals to the narrowest truthful entry surface when that dependency behaves like a boundary.

## Output structure

Resolve `WRITE_ROOT` as the closest git repo root.
Always write generated files relative to `WRITE_ROOT`, even when `SCAN_ROOT` is a deeper subdirectory.

Write to:
- `architecture/output/points.json`
- `architecture/output/links.json`
- `architecture/output/config.json`
- `architecture/output/layout.json` when a guided layout materially improves the render
- `architecture/domains/<domain>.yml` for intermediate per-domain tracking when the map is built incrementally

Create `WRITE_ROOT/architecture/output/` if missing.
Create `WRITE_ROOT/architecture/domains/` if missing when using per-domain tracking files.

Reference material:
- For point and link schemas, output contracts, stable ID patterns, evidence rules, and render tuning, load [references/guidelines.md](./references/guidelines.md).

## Core modeling principle

Model the application as:
- Points: meaningful architectural entities a developer would navigate to directly
- Links: typed relationships between those entities

Do not blindly make every file a point.
Make something a point when it is a stable touchpoint in the architecture, such as:
- A package or module
- A screen or route
- A major view or component
- A view model, controller, store, or state container
- A service, repository, adapter, or gateway
- A domain model or schema root
- A persistence boundary such as a database or cache
- An external system such as an API, SDK, queue, or vendor service

Usually do not make these first-class points unless the user explicitly wants them:
- Tiny helpers
- Formatters
- Extensions
- Small utility functions
- Constants files
- Pure implementation detail types with no architectural role

Those details can be:
- omitted
- folded into a parent point
- surfaced as metadata on a point

## How to walk the codebase

### Step 1 - Find repo root and scope

- Capture the current working directory as `SCAN_ROOT`.
- Determine the closest git repo root and store it as `WRITE_ROOT`.
- Treat `WRITE_ROOT` as the output location for generated files, not as the scan scope.
- Identify whether the user wants the full architecture within `SCAN_ROOT` or a bounded domain within `SCAN_ROOT`.
- Do not assume the repo root is the requested scope. The human is responsible for positioning the working directory before running the skill.
- Enforce a simple path rule: if a candidate file or directory does not live under `SCAN_ROOT`, exclude it unless the user explicitly broadens scope.
- A file outside `SCAN_ROOT` may be inspected only when an in-scope touchpoint directly imports, constructs, invokes, or routes into it, or when a transient dependency must be followed to complete a truthful architectural chain.
- When stepping outside `SCAN_ROOT` for dependency reasons, keep the excursion minimal and evidence-driven. Read only the external files needed to explain the in-scope path, and do not expand that external area into a separate discovery pass.
- If the user runs the skill from `<root>/ios/`, map only the iOS subtree and do not emit Android or other peer-platform traces.
- Default to a full-architecture map for the scanned area under `SCAN_ROOT`.
- If the codebase is large, break the architecture into domain slices and map one slice at a time until the full architecture is covered.
- Do not reduce scope to only "meaningful top-level areas" as a shortcut. Coverage across the full architecture is the default requirement.
- If existing docs or registries reference systems outside `SCAN_ROOT`, treat them as out-of-scope context unless the user explicitly broadens the scan boundary.

### Step 2 - Discover top-level architecture

Before collecting points, identify:
- The dominant architecture patterns or composition styles present in the scanned area
- Top-level packages, apps, modules, and folders
- Entry points such as app bootstrap, main routes, or feature registries
- Major screens or routes
- Primary state containers or orchestration layers
- Data and integration boundaries
- External dependencies that shape the architecture

The point of pattern detection is not to label the codebase for its own sake.
The point is to choose the right collection points for that codebase.
Examples:
- Layered or clean architecture may emphasize use cases, repositories, gateways, and boundary crossings
- MVC, MVVM, MVP, Redux, Elm-style, or Flux-like systems may emphasize controllers, presenters, reducers, stores, selectors, and actions
- Component-driven frontend systems may emphasize routes, layouts, components, hooks, contexts, and client-server boundaries
- Event-driven or workflow-oriented systems may emphasize jobs, handlers, queues, triggers, flows, retries, and state transitions
- Modular monoliths or package-oriented repos may emphasize packages, modules, registries, feature roots, public APIs, and shared infrastructure

For this step, explicitly ask:
- What architectural patterns are actually present?
- Which point types and link types best fit those patterns?
- Which collection points would be missing if you only modeled the obvious top-level files?

Useful things to inspect:
- Package manifests
- App entrypoints
- Navigation or router definitions
- Dependency injection setup
- Feature registries
- Builders, factories, coordinators, assemblers, and composition roots
- Store or state composition
- Service and repository directories
- Network and persistence layers
- Domain models and schema roots
- Background jobs, workers, schedulers, queues, and event handlers
- Hooks, contexts, middleware, interceptors, and composition roots
- Configuration that rewires behavior across environments or feature flags

If beneficial, split discovery across up to 3 sub-agents by independent areas such as:
- feature domains
- architectural layers
- application entrypoints versus data/integration boundaries

Keep final modeling decisions centralized in the main agent.
When the architecture is broad, use the domain slices as the unit of progress and complete them one by one.
Do not assign a sub-agent any area outside `SCAN_ROOT`.

### Step 3 - Extract candidate points

Create candidate points only for entities that matter architecturally.
Favor a richer dataset when the additional nodes and edges clarify the render.
The default failure mode should be under-collapse, not over-collapse.

Good candidates:
- User-visible screens and routes
- Containers, feature roots, and layout shells that own major view composition
- Major views/components that structure a screen
- View models, controllers, stores, presenters
- Builders, factories, coordinators, assemblers, and other composition/orchestration touchpoints
- Services and repositories
- Databases, caches, queues, or APIs
- Packages or modules that contain meaningful feature boundaries
- Use cases, reducers, actions, selectors, middleware, handlers, coordinators, hooks, contexts, registries, and adapters when they materially shape architecture
- Important flows, triggers, rendered states, or helpers when they materially clarify lifecycle, control flow, error handling, or coupling

Weak candidates that usually should not stand alone:
- Tiny helpers
- Mappers with no independent lifecycle
- One-line wrappers
- Small leaf utility files

When in doubt:
- Prefer keeping a candidate if it clarifies stack traversal, domain clustering, or cross-domain coupling
- Collapse or omit only when the candidate is repetitive and does not improve understanding

Dependency boundary rule:
- If a dependency is entered through one stable architectural entry point, prefer one dependency point plus one incoming link from the in-scope caller
- If a dependency exposes multiple distinct entry points that materially change how the app interacts with it, model only those entry points rather than the dependency's internal implementation tree
- Keep dependency-side points and links to the minimum needed to preserve truthful architecture
- Spend detail budget on the app's own composition chain first: entrypoint, dependency assembly, builders/factories/coordinators, containers, screens, meaningful child views, row/item views, and state/orchestration touchpoints

Behavioral nodes are optional and should be used selectively.
Include them when they make the graph more explanatory, not merely more detailed.

Good uses:
- A `flow` point that explains a key lifecycle such as initial load, checkout submit, or sync recovery
- A `trigger` point that clarifies what starts important work
- A `state` point for loading, success, empty, disabled, or error when those states are architecturally important
- A `helper` or `error_handler` point when it materially shapes control flow or coupling

Poor uses:
- Emitting every helper as a point
- Modeling every function as a point
- Creating isolated behavioral nodes that are not anchored to a screen, flow, service, or module

Coverage check for candidate points:
- Do the chosen points let you trace the system from entrypoint to external boundary?
- Is there at least one point for every route, container, screen, and meaningful child view in the scanned area?
- If the codebase uses builder, factory, coordinator, or MVVM patterns, have you represented the builder/factory/coordinator and the view model for each relevant screen flow?
- Do they cover both steady-state dependencies and transient runtime touchpoints?
- Do they expose domain clusters and the important shared infrastructure between domains?
- Are there missing orchestration points such as reducers, actions, handlers, use cases, middleware, contexts, jobs, or schedulers that would make the links more truthful?
- Are there missing boundary points such as caches, queues, SDKs, webhooks, feature flags, configuration registries, or schema roots that would make cross-domain behavior more legible?
- Are there enough intermediate points to make the render legible without forcing a human to infer large hidden jumps?
- Have you traced through enough lower-level components that each important domain path reads as a chain rather than a single coarse edge?
- Have you captured alternate rendered states and lifecycle touchpoints such as loading, empty, disabled, success, retry, and error where those states materially affect navigation, orchestration, or data flow?

When working domain-by-domain, keep an intermediate tracking file for each domain under `WRITE_ROOT/architecture/domains/`.
Recommended filename:
- `WRITE_ROOT/architecture/domains/auth.yml`
- `WRITE_ROOT/architecture/domains/payments.yml`
- `WRITE_ROOT/architecture/domains/shared-infra.yml`

Use these files to track candidate points and candidate links before final normalization.
They exist to make the crawl inspectable by humans and to reduce the chance of losing cross-domain context while moving slice by slice.

Recommended YAML shape:

```yaml
domain: auth
status: in_progress
entrypoints:
  - path: src/auth/routes.ts
    symbol: authRoutes
points:
  - id: route:auth/login
    type: route
    label: LoginRoute
    path: src/auth/routes.ts
    layer: presentation
    status: observed
links:
  - source: route:auth/login
    target: controller:auth/login
    type: owns
    status: observed
shared_links:
  - source: service:auth/session
    target: cache:shared/redis
    type: writes
    targetDomain: shared-infra
    status: observed
notes:
  - Session creation flows into shared Redis cache used by multiple domains.
```

Track shared links explicitly so cross-domain references remain visible while the architecture is being assembled incrementally.

### Step 4 - Extract typed links

For each candidate point, inspect:
- What creates it
- What renders it
- What it calls
- What it depends on
- What state it binds to
- What data sources it reads or writes
- What screen or flow it transitions to
- What lifecycle state changes, loading branches, empty states, retry paths, or modal/navigation triggers it owns

Only create a link if the relationship is meaningful and supported by code.
Walk each important path from top to bottom of the stack wherever possible.
Do not stop at the first obvious dependency hop.
Prefer multiple specific links over a single coarse link when the intermediate architectural steps matter.
Trace through:
- conditional branches
- fallback paths
- feature-flagged behavior
- async triggers and callbacks
- lifecycle hooks and startup paths
- loading, empty, success, disabled, and error state transitions when they change render, navigation, or orchestration
- modal presentation and dismissal paths
- builder, factory, coordinator, and dependency injection assembly paths
- transient dependencies such as helpers, middleware, adapters, or mappers when they materially shape control flow
- cross-domain handoffs and shared infrastructure

The target outcome is not just a bag of local edges.
The graph should reveal domain clusters, full-stack paths through those clusters, and the shared links between domains where those links are real.

Examples:
- `Screen -> ViewModel` as `binds_to`
- `ViewModel -> Service` as `calls`
- `Service -> Repository` as `depends_on`
- `Repository -> Database` as `reads` and `writes`
- `Screen -> Screen` as `navigates_to`
- `Module -> Screen` as `contains`
- `Trigger -> Flow` as `triggers`
- `Flow -> State` as `transitions_to`
- `Service -> Helper` as `uses_helper`
- `Flow -> ErrorHandler` as `handles_error_with`

Granularity pattern:
- Prefer explicit stepwise chains inside the core app when those steps are separate architectural touchpoints
- Do not compress a path like `SceneDelegate -> RootDependencies -> RootBuilder -> RootScreen` into `SceneDelegate -> RootScreen`
- Apply the same rule recursively through tabs, stacks, screens, subviews, row/item views, builders, view models, and conditional branches
- If a path contains four meaningful orchestration steps, expect four points and three links before considering deeper branches
- Collapse only when a step is a trivial pass-through with no independent architectural role

### Step 5 - Normalize and de-noise

Before writing output:
- Merge duplicate entities with the same architectural role
- Remove low-value nodes that only create clutter
- Ensure each point has one clear primary type
- Ensure each link has one clear semantic type
- Ensure point indices are sequential and stable
- Ensure link source and target indices match the point index mapping
- Ensure behavioral nodes attach to a parent screen, flow, service, or module rather than floating as isolated graph noise

## Heuristics for good graphs

Use these heuristics to avoid a bad render:

- Prioritize breadth of architecture over microscopic detail
- Prefer richer architectural granularity over an overly thin first-pass graph
- Prefer full in-app path fidelity over deep dependency expansion
- Keep helper and utility explosion out of the graph only when those helpers do not change control flow, coupling, or stack traversal
- Favor typed relationships over dense generic connectivity
- Prefer one representative point per architectural concept
- Use parent-child containment to preserve context without over-linking
- If one module contains many leaf utilities, keep the module and only include the most important leaves
- Size by importance, not raw file count
- Color by point type or layer
- Use labels for high-importance nodes first
- Let `overview` nodes and edges form the backbone of the graph
- Let `behavior` nodes and edges enrich local understanding without drowning the backbone

## Required workflow

Follow this order:

1. Walk the code to understand architecture
2. Identify the architecture patterns present and select point and link types that fit them
3. Partition the architecture into domain slices when needed so the full map can be built incrementally without dropping coverage
4. For each domain slice, record intermediate candidate points and links in `WRITE_ROOT/architecture/domains/<domain>.yml`
5. Check whether the collection points are sufficient for a faithful point-to-link mapping and add missing categories when needed
6. Trace important flows from top to bottom of the stack, including meaningful branches and transient dependencies
7. Run a completeness audit for each domain slice against routes, containers, screens, child views, orchestrators, state branches, and navigation outcomes
8. Re-audit the domain slice after you think it is complete to search specifically for missed routes, state changes, modals, errors, loading flows, and cross-domain touchpoints
9. Repeat until all relevant domain slices in scope are covered
10. Normalize shared points and cross-domain links across the domain files
11. Output `points.json` and `links.json`
12. Create `config.json` to help render and explore the dataset
13. Create `layout.json` only if a guided layout materially improves readability

Do not skip discovery and jump straight to generation.

## Verification checklist

Before finishing, verify:
- The output folder exists
- The domain tracking folder exists if you used domain slices
- Generated output was written under `WRITE_ROOT`, even if `SCAN_ROOT` was a deeper subdirectory
- No points or links were emitted from sibling or peer directories outside the current working directory subtree unless they were required as direct or transient dependencies of an in-scope touchpoint
- Every emitted `path` starts with or resolves under `SCAN_ROOT`, or is explicitly marked as an out-of-root dependency included to complete an in-scope architectural chain
- `points.json` parses
- `links.json` parses
- `config.json` parses
- `layout.json` parses if created
- Each `architecture/domains/*.yml` file parses if created
- Point indices are sequential and unique
- Every link resolves to valid points
- Every route in scope maps to at least one screen or container path in the graph
- Every meaningful screen or container in scope has its major child views represented
- Every major app flow in scope was traversed step-by-step through composition and orchestration layers rather than being collapsed into coarse jumps
- Every screen flow that uses builder, factory, coordinator, or view-model orchestration includes those touchpoints unless there is clear evidence they are trivial pass-through wrappers
- Dependency points were collapsed to stable entry surfaces where deeper dependency detail would not improve understanding of the core architecture
- Loading, empty, error, retry, modal, and cross-domain navigation branches were checked and represented when architecturally meaningful
- The graph is not overloaded with low-value nodes
- `overview` nodes and edges still form a readable backbone
- Behavioral nodes explain lifecycle, rendering, error handling, or coupling rather than adding incidental detail
- The chosen config reflects the actual fields in the datasets
- Cross-domain links remain explicit rather than being flattened into ambiguous local edges
- A final re-audit was completed after the first full-pass map and either found no material gaps or resulted in additional points and links

## Response expectations

When you complete the work, report:
- Which area of the codebase was mapped
- Whether any out-of-root dependency files were inspected and why
- The files written under `architecture/output/`
- The files written under `architecture/domains/` if any
- The resolved `SCAN_ROOT` and `WRITE_ROOT`
- The modeling decisions that shaped the graph
- Any major inferred areas or confidence limits
