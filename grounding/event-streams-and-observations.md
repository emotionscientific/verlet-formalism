# Streams, Couplings, And Context

> **Dating note.** Written July 2026, before the August 2026 authority recast.
> Where this document says *grant*, the current word is *binding* (an
> operation is attached to a thread by a recorded attach event and absent
> until then); who may attach is policy, not law. The laws and lexicon in
> this repository are current; this document is background and will be
> recast in place.

Status: the core epistemic model for Verlet, grounding tier — it coheres
with [the formalism](../formalism/README.md) and never overrides it. Vocabulary
defers to [the lexicon](../formalism/lexicon.md). The filename keeps its
historical name for link stability; the canonical title is the one above.

The short version:

```text
streams      the only durable truth: append-only, totally ordered events
events       facts; origin is witnessed (the world did it) or discharged
             (a coupling did it, with provenance)
couplings    functions wired into streams: propagators advance the system,
             projections derive, controllers feed back
views        deterministic, regenerable computations over streams; never truth
assembly     the deterministic projection family that compiles context;
             always receipted
surfaces     faithful re-presentations of contracts; they move no data and
             make nothing new happen
receipts     discharged events recording what the runtime actually did
```

## Why This Shape

Verlet must support local app-server use, distributed workers, resumable
threads, publishable operations, and product integration above the kernel.
Context cannot be a prompt builder; it has to be a replayable view over
durable runtime activity.

Streams as the single truth category give us:

- strict temporal order for runtime facts;
- replay, fork, diff, and resume from ordered records;
- deterministic context compilation from known stream positions;
- late interpretation without mutating prior history;
- product memory systems as projections and pipelines, not runtime truth.

The epistemic split is the load-bearing decision: an event documents that
something happened; everything else is a function of events, and the
functions are mutable, versioned, and governed. Truth accumulates;
interpretation is replaceable.

Durability follows from this split. A resumable system does not need to keep a
process heap alive; it needs enough typed boundary facts to reconstruct what
may lawfully happen next. See [Durable Coordination And
Resume](durable-coordination.md) for the operational bridge from these canon
words to activation ledgers, retry policy, leases, and zero-downtime update.

## Truth

### Events And Origin

Events are immutable facts with a stable `kind` and an `origin`:

- **witnessed**: the world or the runtime did it. A user message arrived, a
  process exited, a model call completed, a publish was accepted.
- **discharged**: a coupling produced it. A summary was recorded, a facet was
  extracted, a pipeline was selected. Discharged events carry provenance: the
  source streams, event ranges, and coupling identity (id, version, config
  hash) that produced them.

A discharged event is also called an **observation**. It is a subtype of
event, not a sibling type: the act of interpreting is a fact, and the
interpretation rides on that fact.

If later processing improves on an earlier interpretation, the coupling
discharges a new event that supersedes the old one. The old record stays true
as history:

```text
seq=10 user_message.received                    (witnessed)
seq=11 facet.extracted v1                       (discharged, provenance)
seq=42 facet.extracted v2, supersedes seq=11    (discharged, provenance)
```

The litmus test for filing any record:

```text
Could you delete it and recompute it from events alone?   -> view
Would deleting it lose truth?                              -> event
```

### Receipts Are Events

A receipt is a discharged event documenting a runtime act of resolution or
compilation: a publish, an alias resolution, a grant decision, a context
compile. Receipts are never recomputed; recomputing a receipt under newer code
would lie about history. `compiled_context_receipt` is therefore an event, not
a view, and not a member of a separate store.

### Event Kinds Are Frozen Vocabulary

Event kinds are the addressing scheme for triggers and grants. Every future
controller and every product hook addresses the system through these names.
Renaming an event kind is a breaking ABI change. The V1 release must freeze:

- the kind vocabulary for lifecycle, provider, tool, publish, and context
  events;
- the `origin` field (witnessed/discharged);
- the provenance shape on discharged events.

External inputs enter as witnessed events: chat and SDK messages, CLI
invocations, MCP calls, file arrivals, timer ticks, webhooks, publish and
binding operations.

## Couplings

Every active function in the system attaches through a **coupling**:

```text
source    selector over streams (granted reads)
function  the transformation itself
sink      where output plugs back in (granted writes)
trigger   what activates it
```

Grants live on the coupling. A function has no ambient authority: it reads
what its selector was granted, writes what its sink was granted, and fires
when its trigger says so. The sink is the dangerous leg — write authority on
a stream is authority over truth — so discharges are granted, attributed,
and observed.

Couplings are classified by sink-relation into three roles:

| role | sink-relation | examples |
| --- | --- | --- |
| **propagator** | its own source stream: advances the system | the agent loop: thread -> model -> thread |
| **projection** | elsewhere: derives records or views | summarizer, embedder, facet extractor, indexer |
| **controller** | a control stream: feedback that alters future evolution | pipeline selection, model profile selection, tool gating, compaction triggering |

The lineage is molecular dynamics: the integrator (propagator) advances the
trajectory; estimators and collective variables (projections) compute over
frames; thermostats and metadynamics biases (controllers) are functions of
the trajectory's history coupled back into the dynamics. Agent memory is
metadynamics: history-dependent bias shaping future evolution.

The agent loop itself is the **privileged propagator** — a chaotic coupling
wired thread -> model -> thread, triggered by turn submission. The runtime is
one ontology: streams plus couplings. An agent is a propagator wired into
streams.

### The Feedback Law

A **chaotic** coupling (model-backed, or otherwise not reproducible from its
inputs) cannot produce a view, because there is nothing deterministic to
regenerate. Chaotic output must discharge to a stream as events. A
deterministic coupling's output may instead be a **view**: regenerable,
cacheable, deletable without loss of truth.

This places the determinism boundary in exactly one place:

```text
projection (events -> discharged events)   may be chaotic
assembly   (events + views -> context)     must be deterministic
```

The immutable discharged record absorbs the projection's non-determinism so
that compile can stay pure.

A coupling run emits both kinds of record: witnessed events documenting that
the run happened (model called, tokens spent) and discharged events carrying
the derived payload.

### Quiescence By Budget

Feedback is first-class: discharged events can trigger further couplings.
Classic propagator networks settle because their functions are monotone;
chaotic couplings have no fixpoint, so quiescence cannot be assumed. The
kernel enforces it economically:

- trigger quotas per coupling and per turn;
- propagation depth limits;
- per-turn discharge budgets.

These are layer-0 policy, not suggestions. They are also the governance
answer to runaway agent feedback: bounded inputs, bounded outputs, by law.

## Context

### Pipelines And Sources

The manifest declares context as named **pipelines**, an ordered list where
the first is the default — the same shape as model profiles, and for the same
reason: a future controller selects among declared pipelines; it never
constructs strategies at runtime. The manifest declares the universe; policy
selects within it.

A pipeline is a set of independent **sources**. Each source is:

```text
assembler ref   which assembly function arranges this source's segments
selector        which streams, ranges, kinds, and records it may read
budget share    its slice of the context budget
```

Sources do not know about each other. The kernel performs the deterministic
merge, the final budget fit, and the receipt. Assemblers propose prioritized
segments; the kernel disposes.

```toml
[[context.pipelines]]
id = "default"

[[context.pipelines.sources]]
id = "identity"
assembler = "kernel://assembler/static"
input = "resource://system-prompt"
pinned = true

[[context.pipelines.sources]]
id = "memory"
assembler = "kernel://assembler/record-select"
select = { kind = ["user_preference"], scope = "thread" }
budget_share = 0.1

[[context.pipelines.sources]]
id = "history"
assembler = "kernel://assembler/anchored-window"
select = { stream = "thread", read_plan = "history.default", fallback = "start" }
budget_share = "rest"
```

Pinned sources are not ambient code loads. A system prompt, AGENTS walk,
resource package, or pinned tool contract is selected through an immutable
resource or bind ref, and the compile receipt records the ref, digest,
assembler version, and segment hash.

The history source reads a projection read plan by name. The simplest read plan
is a single raw range from start to the compile frontier. A compaction-aware
plan can keep an early raw prefix, replace a covered range with a discharged
summary, drop a range with a reason, then continue with the raw tail. This keeps
compaction from breaking source independence: the summarizing projection
discharges the summary event, a controller may discharge a named read plan, and
the history source selects from that plan during assembly.

### Assembly Law

Assembly is the deterministic, budgeted projection family that compiles
selected events and views into boundary-visible context, emitting a receipt.
Assembly selects and arranges; it never creates. Anything that needs creating
— a summary, an extraction — is a separate projection whose output discharges
first and gets assembled later.

Determinism over (selected stream positions, read plans, record ids, config,
budget) buys restart determinism, forkability, verifiable receipts, and
prefix-stable provider prompts (which is real prompt-cache money at scale).

The receipt records: stream cursor positions consumed, record ids selected,
read plan refs and resolved frontiers, assembler ids/versions/config hashes,
segments produced, and what was dropped or truncated.

V1 ships kernel built-in assemblers only (static, record-select,
anchored-window), registered behind the assembler contract so that
guest-published assemblers later are a registry entry, not a kernel change.

### Compaction As Couplings

Compaction is not an assembly feature; it is a projection plus an optional
controller:

```text
projection: summarize selected thread range, discharge summary event
controller: budget-threshold policy may discharge context.read_plan.set
assembly:   history source resolves the read plan and emits a compile receipt
```

Creating a summary does not move the history window by itself. The summary is a
discharged event with the compacted text in its payload and provenance over its
covered range. A read plan is the policy fact that says how a later projection
may use raw ranges, discharged-event ranges, and dropped ranges. The context
compile receipt freezes the resolved plan, exact stream cursors, selected event
ids, segment order, and any provider trims for the model request that actually
happened.

### Provider Lowering

Provider-specific packing remains a final adapter pass over the assembled
canonical segments. It may trim, validate, or reject unsupported shapes, and
its trims are recorded in the receipt, but it does not own context policy.

## Stores Are Implementation, Not Ontology

The store vocabulary describes backends, not truth categories:

- `EventStore` / `StreamStore`: append, read, follow ordered records;
  the truth substrate. Backends: SQLite now; Postgres/Turso, S3/R2 segments,
  S2-like systems later.
- Retrieval indexes over discharged events (by kind, scope, embedding):
  views. The existing `ObservationStore` is re-described as such an index;
  the code identifier migrates during the WS-D cleanup, not as a blocking
  rename.
- `ThreadStore`, `ConfigStore`, `ArtifactStore`, `LeaseStore`: unchanged
  roles — runtime lookup state, config, immutable blobs, fencing.

For distributed workers, appends that participate in serialized thread state
include the lease or fencing token. Stream correctness comes from the store,
not from process affinity.

The V1 release-candidate stream envelope, cursor, append, follow, destination
ack, and backend mapping decisions are pinned in
[ADR 0001: Stream Schema V1](https://github.com/emotionscientific/verlet-kernel/blob/main/docs/adr/0001-stream-schema-v1.md).

## Hot-Swap Boundaries

1. **Bindings** swap between turns. A running turn keeps the resolved
   operation snapshot it started with; later turns re-resolve.
2. **Projections and controllers** swap as versioned couplings. A new version
   discharges new events with new provenance; history stays.
3. **Pipelines and read plans** swap per compile, recorded in the receipt, so
   replay can explain why context looked the way it did.

## Implementation Map

| Current surface | Target role |
| --- | --- |
| `VerletAppServer` | Control plane: thread/turn requests, config resolution, provider selection, binding snapshots. |
| `RuntimeHost` | Runtime executor: consumes compiled context, emits provider, tool, bash, and lifecycle events. |
| `SessionStore` / `SqliteSessionStore` | Durable history and resume layer; the migration substrate for stream-backed context. |
| `SqliteMetadataStore` | Control-plane metadata: provider catalog, auth, topology, grants, binding indexes. Not conversation truth. |
| `OperationRegistry` | Resolved operation surface for a turn; the hot-swap truth for runtime operations. |
| Bashkit command surface | PATH-visible shell commands backed by the selected registry; not durable state. |
| `cooldis daemon` | Local resident host for the app-server; owns no context semantics. |

## Current Implemented Slice

Facts about what exists today, annotated against the lexicon:

- `EventStore`, `ObservationStore`, and `RuntimeStore` are runtime contracts;
  `SqliteSessionStore` persists `event_records` and `observation_records`
  beside `session_entries` in `state_home/session_history.sqlite3`.
- Event records carry the frozen V1 kind vocabulary, witnessed/discharged
  origin, and discharged provenance.
- Session entry appends emit `session.entry.appended` events in the thread
  stream with origin derived from the entry role.
- Provider context compilation writes a synchronous
  `context.compile.completed` event plus a `compiled_context_receipt` record
  before the model request. The receipt is currently persisted as an
  observation record; under the lexicon it is an event, and the storage
  re-filing happens with the WS-D identifier migration.
- The receipt records selected session entry ids, output hash, diagnostics,
  provider trim stats, and provenance over the selected pre-compile range.
- App-server `thread/resume` is the public continuation path.
- No public read surface for event/discharged records exists yet.

## Remaining Build Order

Done: the event-kind vocabulary is frozen with `origin` + provenance fields,
and the context compile receipt rides on a discharged event (the
observation-record write stays as a compatibility detail until WS-D migrates
identifiers).

1. Land the pipeline/source manifest schema and the kernel merge + budget
   fit + read-plan-aware receipt path (WS-A), with built-in assemblers only.
2. Compaction as controller + projection behind a kernel-default threshold
   config in runtime defaults; no declared-coupling surface in V1.
3. Defer with shape: guest-published assemblers, declared projections and
   controllers (the coupling registration surface), cross-thread selectors,
   graph views, remote stream backends.
4. Prove restart determinism: run a thread, discharge records, restart,
   recompile the same receipt from stored provenance.

## Test Map

- event append/read survives reopen and preserves sequence order;
- stale lease or fencing tokens cannot append to a serialized stream;
- discharged events require provenance and can supersede older ones;
- the default pipeline reproduces today's context output before any
  compaction logic lands;
- compile receipts survive restart and match selected positions and record
  ids exactly;
- binding changes are visible to a later turn while a running turn keeps its
  snapshot;
- provider auth and secret material never enter event payloads;
- trigger quotas and depth limits halt a deliberately cyclic coupling.

## Non-Goals

- A graph database as a V1 requirement; graphs are views.
- Product memory taxonomy in the kernel; memory is a layer-3 word.
- Mutating historical events to attach new interpretations.
- Hiding coupling side effects inside provider request construction.
- Treating provider-native JSON as canonical context history.
- A general coupling registration surface in V1; the shape ships, the
  registry comes after.
