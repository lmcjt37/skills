# Cosmograph Reference Guidelines

Load this file when you need concrete schema guidance, dataset contracts, ID conventions, or render tuning details while applying the `cosmograph` skill.

## Scan-root discipline

Treat `SCAN_ROOT` as the current working directory captured at skill start.
Treat `WRITE_ROOT` as the closest git repo root.
Do not read, trace, or classify peer-platform or sibling-directory architecture outside `SCAN_ROOT` unless the user explicitly broadens scope.
Do not silently reset scope to the repo root just because it is easier to scan from there.
Exception: you may inspect a file outside `SCAN_ROOT` only when an in-scope route, screen, container, builder, view model, service, or other touchpoint directly or transitively depends on it and you need that file to complete a truthful path through the architecture.
When using that exception, keep the scan narrow and dependency-driven rather than expanding into general discovery of the external subtree.
Always write generated architecture artifacts to `WRITE_ROOT/architecture/...` rather than under `SCAN_ROOT` when those differ.
Do not include out-of-scope architecture in the graph unless it was brought in through that dependency exception or the user explicitly expanded scope.

Examples:
- If `SCAN_ROOT` is `<root>/ios/`, exclude `<root>/android/`
- If `SCAN_ROOT` is `<root>/packages/app/`, exclude `<root>/packages/server/`

## Granularity guidance

Bias toward denser architectural coverage when the added detail improves the render.
Prefer enough intermediate points and links that a developer can follow important paths without mentally reconstructing missing hops.

Good reasons to increase granularity:
- a domain path collapses too many architectural steps into one edge
- cross-domain coupling is real but disappears in an overly coarse graph
- orchestration layers such as hooks, coordinators, reducers, handlers, middleware, or use cases are architecturally important
- builder, factory, assembler, or view-model layers shape how a screen is composed or how dependencies are injected
- route, container, screen, and child-view coverage would otherwise be incomplete
- the app flow includes explicit composition steps such as dependency creation, builder calls, screen initialization, tab wiring, row/item view construction, or nested child-view assembly
- lifecycle and state-driven branches such as loading, empty, modal, retry, or error paths materially affect the flow
- lower-level nodes make clusters and shared infrastructure easier to understand

Poor reasons to increase granularity:
- adding repetitive leaf helpers that do not affect flow or coupling
- modeling every function when the function-level graph adds noise rather than architectural meaning
- expanding inside a dependency boundary when the app only interacts through one or a few stable entry points

Use asymmetric granularity:
- Keep the app's own architecture highly explicit from entrypoint through composition, orchestration, rendering, and navigation
- Keep external or lower-value dependency detail collapsed to the minimum truthful entry surface
- If a dependency has one stable entry point, prefer one dependency point and one incoming link
- If a dependency has multiple meaningful entry points, model those entry points but avoid mapping the dependency's internal tree unless the user explicitly asks for it
- Favor additional points on the app side over additional points inside dependencies when choosing where to spend graph density

Example expectation:
- `SceneDelegate -> RootDependencies -> RootBuilder -> RootScreen` should normally be four points and three links
- Continue the same stepwise treatment through tabs, screens, child views, row/item views, view models, and important conditional branches
- Do not skip intermediate orchestration points unless they are trivial pass-through wrappers with no independent architectural role

## Recommended point taxonomy

Use the smallest useful set of point types that matches the codebase.
Start from these defaults and omit types that do not apply:

- `package`
- `module`
- `container`
- `screen`
- `route`
- `view`
- `component`
- `builder`
- `factory`
- `coordinator`
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
- `modal`
- `helper`
- `error_handler`

When the codebase has meaningful behavioral structure, selectively add these relationship types:
- `triggers`
- `transitions_to`
- `handles_error_with`
- `uses_helper`
- `guards`
- `retries`
- `presents`
- `dismisses`
- `builds`

Useful relationship guidance for composition-heavy codebases:
- Use `creates` when one point initializes another dependency or runtime object
- Use `builds` when a builder, factory, assembler, or coordinator constructs a screen, container, or subtree
- Use `renders` or `contains` to show view composition once the constructed UI subtree is known
- Prefer several typed links across real touchpoints over one vague `depends_on` edge when the app flow is architecturally important

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
  "pointSizeStrategy": "degree",
  "pointClusterBy": "cluster",
  "pointIncludeColumns": ["*"],
  "linkSourceBy": "source",
  "linkSourceIndexBy": "sourceIndex",
  "linkTargetBy": "target",
  "linkTargetIndexBy": "targetIndex",
  "linkColorBy": "type",
  "linkWidthStrategy": "sum",
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
- Default point sizing to `degree` so well-connected nodes surface naturally in the render
- Default link sizing to `sum` with a numeric width column such as `strength` so shared paths accumulate visual weight
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

## Coverage expectations

Treat full architectural coverage as the default requirement for the scanned area, not an aspirational stretch goal.

Minimum expectations:
- at least one point for every route in scope
- at least one point for every container, screen, and meaningful child view in scope
- associated builder, factory, coordinator, and view-model touchpoints represented when those patterns exist
- link chains that let a developer trace from route or entrypoint through presentation and orchestration into data or integration boundaries
- stateful and lifecycle branches checked for loading, empty, error, retry, modal, and cross-domain navigation behavior where present

Before finalizing a graph, run a completeness audit and then a re-audit.
The re-audit should explicitly look for:
- routes discovered from registries that never made it into the dataset
- screens whose child views or containers were collapsed too aggressively
- builders, factories, coordinators, or view models omitted from a screen flow
- modal or navigation triggers that jump into other domains
- alternate state paths that materially change orchestration or rendering

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
