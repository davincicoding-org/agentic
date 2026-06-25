---
name: layer-cake
description: "Use when working in a project organized as the Layer Cake — a one-directional dependency chain (config → storage → models → procedures → store → app → ui) plus an orthogonal axis of cross-cutting code (infra + tools) — whenever you need to decide which layer or namespace a piece of code belongs in, respect the import/dependency direction, or correctly place a static value, data shape, validation, server logic, client state, routing, presentation, a pure utility, an external-service wrapper/integration adapter, or a framework composition/runtime root. Trigger before adding or moving code in such a project, and whenever code smells like it's in the wrong place (data shaping inside a component, business logic in a route, a hand-redefined type that should come from the shared models vocabulary, cache logic in a page, a hardcoded constant or design token that should live in shared config, a 'lib' bucket collecting unrelated cross-cutting helpers, an external-service wrapper or composition root with no clear home). The concrete directory-per-layer mapping and the framework powering each layer come from the project instructions already in context, so this skill stays project-agnostic."
---

# Layer Cake — a dependency spine plus a cross-cutting axis

Some projects are organized as the **Layer Cake**. Its backbone is a strict, one-directional dependency chain — the **spine**:

```
config → storage → models → procedures → store → app → ui
```

But the spine alone can't hold a whole codebase. It is organized by **data-flow position** — each layer is one stage in turning persisted data into pixels. Some code isn't a stage in that flow: a date formatter, a wrapper around a third-party API, the framework bootstrap that wires the app together. That **cross-cutting code lives off the spine, on an orthogonal axis** of two namespaces — `infra` and `tools` (covered below). So the full structure is two things at once: a vertical spine of feature data-flow, and a horizontal axis of cross-cutting code that the spine layers lean on.

This skill is the **conceptual reference** for the Layer Cake: what each layer and namespace is responsible for, what belongs in it, and — most usefully — how to decide where a given piece of code goes. It is deliberately agnostic of *where* things live and *which* frameworks power them. Those bindings are read from the project (see below). The skill names the **roles**; the project names the **places and tools**.

## First: resolve the concrete bindings

The layer names here are abstract. Before placing code, learn how this project realizes them. The **project instructions already loaded in your context** (and any per-layer rule files they point to) define the bindings — you don't need to go fetch them. You're looking for two things per layer:

- **Directory** — which folder is this layer (e.g. some project maps `procedures` to `src/api/`).
- **Framework** — which library powers it (the validation library for models, the server framework for procedures, the client-state library for store, the rendering framework for app/ui, etc.).

Where the project documents a directory→layer map and framework choices, that is your source of truth — follow it over any assumption. If it's missing or ambiguous, ask the user rather than guessing the mapping. Don't carry one project's specifics into your mental model of another; re-resolve per project.

## The spine: one vocabulary, one direction

Four principles make the whole thing hold together. Understanding them lets you place code correctly even in cases this reference doesn't enumerate.

1. **Data shapes are defined once and flow up as a shared vocabulary.** Raw shapes originate in `storage`. `models` derives a clean, domain-named vocabulary from them. *Every layer above types and validates against that vocabulary* — it never re-declares a data shape that already exists below. When you find yourself writing a fresh type for data that already lives in models, that's the signal to derive instead.

2. **Dependencies flow strictly downward.** A layer may import only from layers to its left in the chain. `ui` may use `models`; `models` may never reach into `ui`. This is what keeps concerns from bleeding: a presentational component physically *can't* hold a database query, because it can't import that far down the chain. A dependency pointing the wrong way means the code is in the wrong layer.

3. **Concrete values are defined once and read by anyone.** Where the data *shapes* originate in `storage`/`models`, the data *values* — design tokens, enum option-lists, taxonomies, static constants — originate in `config`, the dependency-free base. It sits left of everything, so any layer may read it: `models` derives an enum schema from a config option-list, `ui` themes from config tokens. Distinguish the axes — `config` holds *what the value is*; `models` holds *what the shape is*. A literal constant or token duplicated at a use site is the signal to lift it into config.

4. **A layer's multiplicity follows its role.** The layers that *originate* a source-of-truth every branch consumes — `config` (values), `storage` (shapes), `models` (vocabulary) — are **singletons**: exactly one canonical set, because forking them forks the spine. The *consumer* layers — `store`, `app`, `ui` — **fan out, one instance per deployable**, each reusing/extending the shared layers above, never re-originating them. `procedures` is the hinge: **one band per server entry point** — split it when you add a *server* (a second backend gets its own `procedures`), never when you add a *client*, since every client shares the one contract. The singleton↔fan-out line falls exactly on the originate→consume boundary; the folder structure below realizes it.

## Off the spine: cross-cutting code (the orthogonal axis)

A naive read of the chain is just `storage → models → procedures → store → app → ui` — data becoming pixels. That bare chain is **missing three things**, and naming them is the whole point:

- **`config`** — the *values* base, left of `storage` (already covered above).
- **`infra`** — the composition/runtime root: the one node where **upward imports are legal**.
- **`tools`** — the leaf sidecar: pure utilities, wrappers, and integration adapters, **strictly downward**.

`config` joins the spine (it's a position — the base). `infra` and `tools` do **not**: they're the orthogonal axis.

### Why an axis, not more layers

A spine layer is defined by its *position* in the data flow. Cross-cutting code has no single position, and forcing it onto the spine fails in two opposite directions:

- **Leaf utilities** (a date formatter, a string helper, a thin API wrapper) want to sit *far-left* — everyone imports them, they import almost nothing. But they aren't data values (`config`) or data shapes (`models`), so no spine slot fits.
- **Composition roots** (the framework bootstrap that mounts your server router into your client, or wires auth callbacks to email templates) import *across the whole chain* — they depend in **both** directions at once. No single position can hold something that reaches both left and right.

Collecting all of it into one `lib` bucket is the trap: you'd put strictly-downward leaf code and upward-reaching bootstrap code under **one rule** — but their import constraints are *opposite*, so no single rule can be right. The bucket becomes unruleable.

The fix is to split cross-cutting code by its real differentiator — **reason to change** — into two namespaces:

- **`infra` — the composition/runtime root.** Changes when the *framework wiring* changes. It bootstraps a framework against the whole app, so it necessarily imports upward; this is the **only** place upward imports are sanctioned. It exists precisely *because* the bootstrap paradox makes a single spine position impossible. Keep it to wiring — no business logic grows here.
- **`tools` — leaf code.** Changes when an *external dependency* or a *helper's implementation* changes. Pure utilities, thin third-party wrappers, and integration adapters. **Strictly downward**, like any spine layer.

### Namespaces, not layers

Neither `infra` nor `tools` has a single spine position, so neither carries one blanket rule. Each is a **namespace**: a set of named subfolders (`tools/date`, `tools/google`, `infra/trpc`, `infra/auth`…), and **the rule attaches per subfolder** based on what that subfolder imports. What *is* uniform: the **downward-import law still governs every edge** — within `tools`, out of `tools`, and out of `infra`. The single exception in the entire system is `infra`'s composition roots reaching upward, and that exception is the reason `infra` is its own namespace rather than part of `tools`.

### The integration contract holds wherever it lands

A wrapper around a foreign service keeps its **foreign/wire shapes private and maps them to `models` at the boundary** — so every layer above stays integration-agnostic. This is the same contract `procedures` follows (see below), and it is **location-independent**: it holds whether the adapter sits in `tools` (a standalone vendor wrapper) or gets consumed inline in `procedures` (a webhook payload). Foreign shapes never surface above the boundary that owns them.

### Reject two impostors

Some code *looks* cross-cutting — "just a shared helper" — but belongs on the spine. Catch these before they pollute `tools`, or it slides back toward an unruleable bucket:

- **A static value** (a constant, design token, option-list) is not a tool — it's a **value**. It belongs in **`config`**. The tell: it's data, not behavior.
- **A helper that reads storage or encodes a business rule** is not a tool — it's **business logic**. It belongs in **`procedures`** (under the relevant feature). The tell: it imports `storage` to touch domain rows, or it decides *what the business does*.

### Deciding where cross-cutting code goes

Walk it in order; the first match wins:

1. Is it a static **value** (constant, token, option-list)? → **`config`**.
2. Does it **read storage** or encode a **business rule**? → **`procedures`** (the relevant feature).
3. Does it **wire a framework** against the app and necessarily **import upward**? → **`infra`** (the right subfolder).
4. Otherwise it's **leaf code** — a pure util, a thin wrapper, an integration adapter → **`tools`** (the right subfolder, strictly downward).

## The layers

### config — concrete static values
The base of the cake and the single source of truth for *values* (as distinct from shapes): design tokens (colors, spacing, radii, type scale, motion), shared reference data (enum option-lists, taxonomies, fixed lookups), and app-wide static constants (limits, defaults). Pure structured data — **no logic, no IO, no data shapes, no framework imports** (no React/DOM/native renderer), so *any* layer above and *any* framework can read the same source. Depends on nothing; sits left of `storage`. This is **not runtime environment** — it holds static build-time data, never secrets or per-environment URLs (those are deployment config, a different concern the name invites confusing with). Nor is it **dynamic/runtime config** despite the shared word: a feature flag or per-tenant setting read at runtime is *data* — its shape lives in `storage`, flows up via `models`/`procedures` like any other row. Only its allowed keys/option-list and build-time defaults live here. The `config`↔`models` seam: config exposes the literal allowed values (`SPECIES = ['dog','cat',…]`); models derives the validated shape from them (`species = z.enum(SPECIES)`). Values live once here; shapes derive upward — never redeclare the option-list in models.

### storage — persisted data structure
The source of truth for raw data shapes across **every persisted store** — relational tables, but equally event streams, search-index documents, blobs, and snapshots. Defines those shapes and nothing else: no queries, no access control, no transformation, no application logic. A store whose contents are *derived* from another (an index, a projection, a cache, an audit/event log) still declares its shape here — the transform that fills it lives in `procedures`, not here. Must stay **import-safe for clients** — it cannot pull in persistence drivers or connection code, so that `models` can build on it from anywhere (server or client). Everything above derives its shapes, directly or transitively, from here.

### models — the shared data vocabulary
The single source of truth for *what data is* across server and client. Schemas/types **derived** from storage shapes, named for the **domain concept**, not the layer or usage (avoid `Input`/`Response`/`Payload`-style suffixes — name what the data *is*). Compose from reusable atoms; split by domain concept, not by consumer or read/write pattern. Never hand-roll a shape that already exists below — derive it. Within models, dependencies flow top-to-bottom; shared atoms live in the earliest section that makes domain sense. This layer is the contract every layer above types against.

### procedures — server-side request and event handling
Handles user actions and server-originated events: defines the request handlers (queries and mutations), **plus any long-lived subscription or stream that emits data over time** (not every entry point is a one-shot request/response), enforces access control, and **shapes raw inputs — stored rows *and* external payloads alike — into clean model types at the boundary**. Inputs are validated against model schemas; values returned *or emitted* are model types or compositions of them — no locally defined types *cross upward*. Integration handled inline here follows the integration contract: foreign/wire contract shapes (webhook payloads, third-party request/response shapes) are defined and consumed **privately**, never surface above the boundary, and are mapped to/from models before anything leaves — so every layer left of `procedures` stays integration-agnostic. (A *standalone* vendor adapter that isn't tied to one request lives in the `tools` axis instead, under the same contract — see "the integration contract holds wherever it lands".) A locally-defined shape is fine *as long as it stays inside* `procedures`; the "no local types" rule constrains what flows up, not internal boundary parsing. This is the only place raw inputs — a stored document or a foreign payload — become model-shaped, whether returned from a request or pushed over a subscription; once data leaves here it's already clean for every layer above.

### store — client-side state
Wraps the server's data interface into domain-specific client interfaces. Owns read orchestration — whether data is **fetched on demand or streamed in** — plus caching, cache invalidation, optimistic updates, mutations, and the lifecycle of any live subscription (connect, resubscribe, reconcile pushed updates into the cache). Organize **one module per domain** (mirror the server structure), not one god module. Expose **hooks/interfaces**, not raw server calls, so the app layer consumes a clean domain API. Prefer updating the cache directly over invalidating-and-refetching when the result can be reconstructed from a mutation's input/response or a pushed event — a refetch costs a round-trip. No UI, no JSX, no rendering. Types come from models.

### app — routing and wiring
Connects store and procedures to ui, and owns routing. Server-rendered entry points fetch data via procedures directly; client entry points consume store hooks — never raw queries in a page. Any server-action / form-action wrappers stay **thin**: forward to a procedure, add route-scoped context (an id from the URL), nothing more. No business logic, no data transformation, no data shapes redefined here — if the shape doesn't fit, fix it down in procedures or store. This layer is glue.

### ui — presentation
Isolated, self-contained components that receive **all** data through props typed from models (directly, or narrowed via picks/indexed access — never widened, aliased, or redefined). No server imports. No data shaping, validation, flattening, or digging into raw relations — a value should arrive as a ready-to-render prop. Prefer the project's UI primitives over raw elements. Callback props that carry structured data type their parameters from models too.

## Folder structure

One cake per **project** — the vocabulary (`config`/`storage`/`models`) is defined once and shared. The folder names below are the layer *roles*; use the project's real directory bindings (see "resolve the concrete bindings").

### Single deployment — one folder per layer
Everything is one deployable unit holding every layer. One build; imports flow strictly downward. The spine is the vertical chain; `infra` and `tools` are the orthogonal sidecars the spine leans on (each a namespace of subfolders, not a single positioned layer).

```
<repo root>
  config/           # spine — values base
  storage/          # spine
  models/           # spine
  procedures/       # spine
  store/            # spine
  app/              # spine
  ui/               # spine
  infra/            # axis — composition/runtime roots (upward imports legal here only)
    <bootstrap-a>/  #   e.g. the server-router → client wiring
    <bootstrap-b>/
  tools/            # axis — leaf utilities, wrappers, integration adapters (strictly downward)
    <util-a>/
    <vendor-x>/     #   e.g. a third-party service adapter
```

### Multiple deployments — shared layers at root, deployables under `apps/`
The root keeps the shared (optional) version of every layer. Each deployable lives under `apps/` and may define its **own slice of any layer that reuses/extends the root one of the same kind** (compose, wrap, re-export, add app-specific pieces) — not a bare import.

```
<repo root>
  config/  storage/  models/        # vocabulary — always shared
  procedures/  store/  app/  ui/    # optional shared layers
  infra/  tools/                    # cross-cutting axis — shared at root like the vocabulary
  apps/
    backend-a/
      procedures/        # reuses/extends root procedures/
    backend-b/
      procedures/
    web/
      store/  app/  ui/  # each reuses/extends its root counterpart
```

Rules:
- A layer needed project-wide stays at **root**; a slice owned by one deployable lives in **`apps/<unit>/`** and reuses/extends its root counterpart.
- Direction still flows downward through every seam — `apps/web/store` → root `store` → root `procedures` → root `models`. A seam realized over the wire (an `apps/web` calling root/another deployable's `procedures` via RPC) is the same downward edge, typed through the shared `models`.
- **No app reaches into another app.** A sibling deployable at the same layer is consumed as an external contract (mapped to `models` privately in `procedures` — see the integration rule there).

## Which layer or namespace does this belong in?

| The work | Where |
|----------|-------|
| A design token, enum option-list, taxonomy, or static constant (a *value*, not a shape) | **config** |
| A new persisted field, table, event, index doc, or entity shape | **storage** |
| A data shape used in more than one place, or a validation schema | **models** |
| Handling a request; a query/mutation; emitting a subscription/stream; access control; turning stored data into model shapes; a helper that reads storage or encodes a business rule | **procedures** |
| Client-side fetching or subscribing, caching, invalidation, optimistic updates, mutations | **store** |
| A route/page; wiring data into components; a thin server/form action | **app** |
| Anything the user sees — rendering, layout, presentation | **ui** |
| A framework composition/runtime root that bootstraps the app and must import upward | **infra** (axis) |
| A pure utility, a thin third-party wrapper, or a standalone integration adapter — strictly downward | **tools** (axis) |

## Wrong-layer smells

These mean code has landed in the wrong place — move it toward the layer noted, or reshape so the data arrives correctly:

- A **component** validating, flattening, or asserting into raw/nested data → shape it in **procedures**; let it arrive as a ready prop.
- **Business logic** in a route, page, or server action → push down to **procedures**.
- A **data type redefined** at its use site → derive it from **models**.
- **Cache invalidation / refetch logic** in a page or component → it belongs in **store**.
- A **query or direct data access** inside a page component → route it through **procedures** (server) or **store** (client).
- A **storage schema file importing a driver/connection** → breaks client-safety; keep storage import-safe so models can build on it.
- A **hardcoded hex, magic constant, or design token** baked into a component, theme, or schema → lift the value into **config**.
- An **option-list redeclared** in two places → put the values in **config**; derive the shape from them in models.
- **Logic, a data shape, or a framework/React import inside config** → config is values-only; move the shape to models, the logic up its proper layer.
- **Runtime secrets or per-environment URLs in config** → that's deployment environment, not the config layer's static build-time data.
- A **foreign/external contract shape** (webhook payload, third-party response) surfacing **above** the boundary that owns it — leaking out of `tools` or `procedures` into `models`, `store`, or `ui` → keep it private to the adapter; map it to a model at the boundary so everything above stays integration-agnostic.
- A **`lib`/`utils`/`shared` bucket** collecting unrelated cross-cutting code under one rule → split by reason-to-change: framework bootstrap (upward) → **infra**; leaf utils/wrappers/adapters (downward) → **tools**; and pull out the two impostors below.
- A **static value living in a "util" module** (a constant, token, option-list dressed up as a helper) → it's a value; lift it into **config**.
- A **"helper" that imports `storage` or encodes a business rule** sitting in `tools`/`lib` → it's business logic; move it to **procedures** under its feature.
- A **`tools` (leaf) module importing upward** — reaching into `procedures`, `store`, `app`, or `ui` → either it's misplaced, or it's actually a composition root and belongs in **infra** (the only place upward imports are legal).
- **Business logic growing inside an `infra` composition root** → infra is wiring only; the logic belongs down in **procedures**.
- A **wrong-direction import** (a lower layer reaching up), anywhere outside an `infra` composition root → the strongest signal of all that something is misplaced.
