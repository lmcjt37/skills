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

1. Never write outside the closest git repo root.
2. Start from the current working directory. Treat it as the requested scan root unless the user explicitly broadens scope.
3. Start with discovery. Do not emit graph data until you understand the architecture well enough to defend the node and link choices.
4. Stay architecture-pattern agnostic. Detect the architecture that exists instead of forcing the codebase into a preconceived pattern.
5. Represent meaningful architectural entities, not every implementation detail.
6. Every link must have semantic meaning. Avoid generic unlabeled edges.
7. Prefer evidence over interpretation. Mark inferred edges or classifications explicitly.
8. Keep the graph renderable. If a choice would create noise without insight, collapse or omit it.
9. Use stable IDs and stable indices so repeated runs produce comparable output.
10. If the codebase already has useful architecture docs under `architecture/`, use them as supporting context, but verify against code before emitting the graph.
11. If layout is obvious from the graph shape, omit `layout.json`. Only create it when it materially improves readability.
12. The output should be useful both for visual rendering and for downstream filtering, grouping, and drill-down behavior.
13. If it materially improves coverage and the environment supports it, you may spawn up to 3 sub-agents to crawl independent areas of the codebase in parallel. Use them to accelerate evidence gathering, not to bypass synthesis.

## Output structure

Write to:
- `architecture/output/points.json`
- `architecture/output/links.json`
- `architecture/output/config.json`
- `architecture/output/layout.json` when a guided layout materially improves the render
- `architecture/domains/<domain>.yml` for intermediate per-domain tracking when the map is built incrementally

Create `architecture/output/` if missing.
Create `architecture/domains/` if missing when using per-domain tracking files.

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

## Recommended point taxonomy

Use the smallest useful set of point types that matches the codebase.
Start from these defaults and omit types that do not apply:

- `package`
- `module`
- `screen`
- `route`
- `view`
- `component`
- `view_model`
- `controller`
- `store`
- `service`
- `repository`
- `model`
- `database`
- `cache`
- `api`
- `job`
- `external_system`

Each point should include enough metadata to support filtering and styling.

Recommended point shape:

```json
{
  "id": "screen:home",
  "index": 0,
  "label": "HomeScreen",
  "type": "screen",
  "parentId": "module:home",
  "path": "Sources/Home/HomeScreen.swift",
  "symbol": "HomeScreen",
  "layer": "presentation",
  "domain": "home",
  "cluster": "home",
  "subcluster": "ui",
  "technology": "swiftui",
  "platform": "ios",
  "graphLevel": "overview",
  "importance": 0.92,
  "sizeWeight": 0.92,
  "status": "observed",
  "notes": "Primary landing screen"
}
```

### Required point fields

- `id`: stable string identifier
- `index`: unique sequential integer starting at `0`
- `label`: human-readable label
- `type`: one of the selected point types

### Strongly recommended point fields

- `parentId`: containing module or package
- `path`: primary source file path
- `symbol`: main symbol name
- `layer`: architectural layer such as `presentation`, `state`, `domain`, `data`, `integration`
- `domain`: bounded feature area
- `cluster`: major deterministic placement group, usually the same as `domain`
- `subcluster`: local grouping within the domain such as `ui`, `flow`, `state`, `service`, `helper`, or `external`
- `graphLevel`: usually `overview` or `behavior`
- `importance`: relative weight for sizing or prioritization
- `sizeWeight`: numeric hint for rendering or filtering when `importance` is categorical or less precise
- `status`: `observed` or `inferred`

## Recommended link taxonomy

Every link must express a clear relationship.
Start from these defaults and use only the relationships you can justify from code:

- `contains`
- `renders`
- `owns`
- `binds_to`
- `navigates_to`
- `calls`
- `depends_on`
- `reads`
- `writes`
- `publishes`
- `subscribes_to`
- `creates`
- `maps_to`
- `uses`

When the codebase has meaningful behavioral structure, selectively add these point types:
- `flow`
- `trigger`
- `state`
- `helper`
- `error_handler`

When the codebase has meaningful behavioral structure, selectively add these relationship types:
- `triggers`
- `transitions_to`
- `handles_error_with`
- `uses_helper`
- `guards`
- `retries`

Recommended link shape:

```json
{
  "id": "screen:home->view_model:home",
  "source": "screen:home",
  "sourceIndex": 0,
  "target": "view_model:home",
  "targetIndex": 1,
  "type": "binds_to",
  "weight": 0.9,
  "strength": 0.9,
  "direction": "forward",
  "flow": "initial-load",
  "graphLevel": "overview",
  "status": "observed",
  "evidence": "HomeScreen initializes and observes HomeViewModel",
  "path": "Sources/Home/HomeScreen.swift"
}
```

### Required link fields

- `id`: stable string identifier
- `source`: source point id
- `sourceIndex`: source point index
- `target`: target point id
- `targetIndex`: target point index
- `type`: semantic relationship type

### Strongly recommended link fields

- `weight`: relative strength or visual emphasis
- `strength`: numeric hint for prominent versus secondary relationships
- `flow`: named lifecycle, user journey, or process this edge belongs to
- `graphLevel`: usually `overview` or `behavior`
- `status`: `observed` or `inferred`
- `path`: source file where the relationship was verified
- `evidence`: short factual note explaining the relationship

## How to walk the codebase

### Step 1 - Find repo root and scope

- Start from the current working directory and treat it as the scan root.
- Determine the closest git repo root.
- Identify whether the user wants a whole-app map or a bounded domain.
- Do not assume the repo root is the requested scope. The human is responsible for positioning the working directory before running the skill.
- Default to a full-architecture map for the scanned area.
- If the codebase is large, break the architecture into domain slices and map one slice at a time until the full architecture is covered.
- Do not reduce scope to only "meaningful top-level areas" as a shortcut. Coverage across the full architecture is the default requirement.

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

### Step 3 - Extract candidate points

Create candidate points only for entities that matter architecturally.

Good candidates:
- User-visible screens and routes
- Major views/components that structure a screen
- View models, controllers, stores, presenters
- Services and repositories
- Databases, caches, queues, or APIs
- Packages or modules that contain meaningful feature boundaries
- Important flows, triggers, rendered states, or helpers when they materially clarify lifecycle, control flow, error handling, or coupling

Weak candidates that usually should not stand alone:
- Tiny helpers
- Mappers with no independent lifecycle
- One-line wrappers
- Small leaf utility files

When in doubt:
- Prefer collapsing a low-value candidate into metadata on a parent point
- Or omit it entirely if it adds graph noise

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
- Do they cover both steady-state dependencies and transient runtime touchpoints?
- Do they expose domain clusters and the important shared infrastructure between domains?
- Are there missing orchestration points such as reducers, actions, handlers, use cases, middleware, contexts, jobs, or schedulers that would make the links more truthful?
- Are there missing boundary points such as caches, queues, SDKs, webhooks, feature flags, configuration registries, or schema roots that would make cross-domain behavior more legible?

When working domain-by-domain, keep an intermediate tracking file for each domain under `architecture/domains/`.
Recommended filename:
- `architecture/domains/auth.yml`
- `architecture/domains/payments.yml`
- `architecture/domains/shared-infra.yml`

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

Only create a link if the relationship is meaningful and supported by code.
Walk each important path from top to bottom of the stack wherever possible.
Do not stop at the first obvious dependency hop.
Trace through:
- conditional branches
- fallback paths
- feature-flagged behavior
- async triggers and callbacks
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
- Keep helper and utility explosion out of the first-pass graph
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
4. For each domain slice, record intermediate candidate points and links in `architecture/domains/<domain>.yml`
5. Check whether the collection points are sufficient for a faithful point-to-link mapping and add missing categories when needed
6. Trace important flows from top to bottom of the stack, including meaningful branches and transient dependencies
7. Repeat until all relevant domain slices in scope are covered
8. Normalize shared points and cross-domain links across the domain files
9. Output `points.json` and `links.json`
10. Create `config.json` to help render and explore the dataset
11. Create `layout.json` only if a guided layout materially improves readability

Do not skip discovery and jump straight to generation.

## Output contracts

### `architecture/output/points.json`

Must be a JSON array of points.

Requirements:
- Every point has a unique `id`
- Every point has a unique sequential `index`
- Indices start at `0`
- Labels are concise and readable
- Types are consistent across the dataset

### `architecture/output/links.json`

Must be a JSON array of links.

Requirements:
- Every link has a unique `id`
- `source` and `target` match point ids
- `sourceIndex` and `targetIndex` match point indices
- Relationship types are explicit and finite
- Avoid duplicate links unless the relationship types differ meaningfully

### `architecture/output/config.json`

Must contain the Cosmograph data accessors and recommended rendering defaults for this dataset.

Base template:

```json
{
  "pointIdBy": "id",
  "pointIndexBy": "index",
  "pointLabelBy": "label",
  "pointColorBy": "type",
  "pointSizeBy": "sizeWeight",
  "pointClusterBy": "cluster",
  "pointIncludeColumns": ["*"],
  "linkSourceBy": "source",
  "linkSourceIndexBy": "sourceIndex",
  "linkTargetBy": "target",
  "linkTargetIndexBy": "targetIndex",
  "linkColorBy": "type",
  "linkWidthBy": "strength",
  "linkIncludeColumns": ["*"],
  "showDynamicLabels": true,
  "showTopLabels": true,
  "showTopLabelsLimit": 25,
  "showClusterLabels": true,
  "showFocusedPointLabel": true,
  "selectPointOnClick": "single",
  "focusPointOnClick": true,
  "fitViewOnInit": true,
  "fitViewDelay": 0,
  "simulationDecay": 3000,
  "repulsion": 0.5,
  "gravity": 0.1
}
```

Adjust the config to the dataset:
- Cluster by `cluster`, `layer`, `domain`, or `parentId` depending on which gives the clearest graph
- Color by `type` unless `layer` is more informative
- Size by `sizeWeight`, `importance`, or another normalized weight
- If you already provide manual coordinates via layout, consider disabling or softening simulation

### `architecture/output/layout.json`

Create this only when the graph benefits from guided positioning.

Good reasons to create it:
- The application has strong left-to-right architectural layers
- There are clear vertical lanes by domain or module
- The dataset is dense enough that default simulation produces an unhelpful hairball

Prefer deterministic cluster-level placement over hand-positioning every point.
The default goal is stable domain or module placement across runs, with simulation handling the local arrangement.

Recommended shape:

```json
{
  "pointPositions": {
    "screen:home": [-0.9, 0.2],
    "view_model:home": [-0.4, 0.2],
    "service:user": [0.2, 0.2],
    "repository:user": [0.6, 0.2],
    "database:primary": [0.9, 0.2]
  },
  "clusterPositionsMap": {
    "presentation": [-0.8, 0],
    "state": [-0.3, 0],
    "domain": [0.2, 0],
    "data": [0.7, 0],
    "integration": [1.0, 0]
  }
}
```

If you emit point-level coordinates, ensure the render config uses the corresponding fields when the consuming code expects them.
If the consuming setup only supports cluster guidance, emit cluster positions instead.

Recommended layout guidance:
- Use `clusterPositionsMap` keyed by `cluster` or major domain
- Keep placement strategies stable across runs of the same codebase
- Prefer readable left-to-right lanes, grid sectors, or hub-and-spoke placement based on repo shape
- Keep core `overview` nodes closer to the cluster center
- Allow `behavior` nodes to sit slightly more peripherally through moderate repulsion and link distance
- Keep force tuning moderate; the goal is readable domain interiors, not exploding related nodes far apart
- Optionally support `pointClusterStrengthBy` when some point types should stay closer to the cluster center than others

Recommended optional layout fields:

```json
{
  "pointClusterBy": "cluster",
  "clusterPositionsMap": {
    "auth": [-1200, 200],
    "home": [-200, 200],
    "payments": [900, -100],
    "settings": [900, 700]
  },
  "simulationClusterStrength": 0.65,
  "simulationRepulsion": 0.25,
  "simulationLinkDistance": 3,
  "strategy": "major-domains-grid"
}
```

## Stable ID guidelines

Point IDs should be deterministic and composable.

Preferred patterns:
- `package:<name>`
- `module:<name>`
- `screen:<name>`
- `view_model:<name>`
- `service:<name>`
- `repository:<name>`
- `database:<name>`
- `api:<name>`

If names collide, namespace by domain or module:
- `screen:settings/profile`
- `service:auth/token-refresh`

Link IDs should also be deterministic:
- `<sourceId>-><targetId>:<type>`

## Evidence and inference

Include `status` on points and links:
- `observed` when verified from code
- `inferred` when derived from structure, naming, or conventions

Use `evidence` on links and optionally on points for a short factual note.

Examples:
- `observed`: route registry imports `SettingsScreen`
- `inferred`: `UserService` classified as `service` based on directory and public methods

Do not hide uncertainty.

## Render-first guidance

Optimize the dataset for an actually useful render:

- A developer should be able to visually distinguish presentation, state, domain, data, and integration areas
- Important screens and orchestrators should be easy to spot
- Cross-domain edges should stand out from local containment edges
- Navigation edges should read differently from dependency edges
- Labels should remain readable for important points without covering the whole canvas
- Behavioral details should clarify local execution without obscuring the overall architecture

Recommended defaults:
- Use color for `type` or `layer`
- Use size for `sizeWeight` or `importance`
- Use width for `strength` or `weight`
- Use clustering for `cluster`, `layer`, `domain`, or `parentId`
- Use `contains` links sparingly if they visually overwhelm operational relationships

## Verification checklist

Before finishing, verify:
- The output folder exists
- The domain tracking folder exists if you used domain slices
- `points.json` parses
- `links.json` parses
- `config.json` parses
- `layout.json` parses if created
- Each `architecture/domains/*.yml` file parses if created
- Point indices are sequential and unique
- Every link resolves to valid points
- The graph is not overloaded with low-value nodes
- `overview` nodes and edges still form a readable backbone
- Behavioral nodes explain lifecycle, rendering, error handling, or coupling rather than adding incidental detail
- The chosen config reflects the actual fields in the datasets
- Cross-domain links remain explicit rather than being flattened into ambiguous local edges

## Response expectations

When you complete the work, report:
- Which area of the codebase was mapped
- The files written under `architecture/output/`
- The files written under `architecture/domains/` if any
- The modeling decisions that shaped the graph
- Any major inferred areas or confidence limits
