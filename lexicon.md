# Verlet Lexicon

Status: canonical internal vocabulary, part of [the formalism](README.md).
Other docs defer to this one. When a doc or identifier disagrees with the
lexicon, the doc or identifier is wrong. The laws that bind the system as a
whole — the Split, the Three Laws, the Ladder — live in [laws.md](laws.md);
every word below carries its own.

Names are load-bearing in Verlet. Event kinds are trigger addresses. Stream
names are authority scopes. Operation names become shell commands. Aliases resolve
to immutable hashes. A name is a control surface, so an imprecise word is an
imprecise authority boundary. This file exists to keep the words exact.

This is the kernel's own language. User-facing jargon (subagents, memory,
hooks, skills) is a separate, later dictionary; every public word must lower
to lexicon words before it reaches kernel APIs, manifest schemas, or event
kinds. See "Public Jargon" below.

Rules of the lexicon:

- one word per concept, one concept per word;
- every word carries a law;
- banned words list their replacement;
- new primitives must be named here before they are implemented.

## Truth

### stream

An append-only, totally ordered sequence of events owned by a scope (thread,
tenant, control). The only durable truth in the system.

Law: streams are never mutated, only appended. Order is authoritative.

### event

An immutable record of a fact. Carries a stable `kind` name and an `origin`.

Law: events are never edited or deleted. Later events may supersede an
interpretation; nothing supersedes history.

Event kinds are frozen vocabulary: they are the addressing scheme for triggers
and selectors. Renaming an event kind is a breaking ABI change.

### witnessed

Event origin: the world or the runtime did it. A user message arrived, a
process exited, a model call completed, a publish was accepted. Witnessed
events have no producing function; their authority comes from the runtime
attesting to them.

### discharged

Event origin: a coupled function produced it. A summary was recorded, a facet
was extracted, a pipeline was selected.

Law: no discharge without provenance.

### observation

A discharged event. Not a parallel record category and not a second store: an
observation is an event whose origin is a coupling. The act of interpreting is
a fact; the interpretation rides on that fact.

Replaces: `Observation` as a sibling type of `Event`, and `ObservationStore`
as a second category of truth. A store may still index discharged events for
retrieval; that index is a view.

### provenance

The source streams, event ranges, and coupling identity (id, version, config
hash) that produced a discharged event.

Law: provenance is sufficient to explain a discharged event and locate every
input it consumed.

### receipt

A discharged event documenting a runtime act of resolution or compilation: a
publish, an alias resolution, a policy decision, a context compile. The runtime
component that performed the act is named in `discharged_by`, and the
function/version it ran in `function`; the receipt's provenance is what makes
the act auditable.

Law: receipts record what the runtime actually did. They are never recomputed;
recomputing a receipt under newer code would be a lie about history.

Note: `compiled_context_receipt` is therefore an event, not an observation.

### residue

Model-run byproduct addressed to the producing model family's own future
turns: reasoning traces, provider-side compaction state, encrypted reasoning
items. Semantically it is a record of nothing — no consumer other than the
producing family may treat it as fact — but it can be generationally
load-bearing: families trained in preserved-reasoning mode (Kimi K3,
2026-07-16) require it back verbatim for stable continuation. Residue is
what a run leaves behind, not what it produced; distinct from the
specialization ancestry's *residual program*, which is produced output with
a receipt. Named 2026-07-16 ahead of implementation per the naming law,
anchor-ratified same day
(tool expression and context portability design note, July 2026, D8/D9 addendum).

Law: residue routes by readability. Readable residue enters the stream on
the model-turn event that produced it, tagged with the producing model
profile; assembly replays it verbatim only into context compiled for the
same family (the compatibility scope the profile declares), and into any
other family's context it is re-presented as quoted content or omitted, per
the pipeline's declaration — the assembly receipt records the disposition.
Unreadable residue cannot discharge, so under Law 2 it is never truth and
never a view: it is cache, may hold only stream-derivable content, and dies
at every rebind and thread end.

## Couplings

### coupling

The wiring declaration for every active function in the system:

```text
source    selector over streams (declared reads)
function  the transformation itself
sink      where output plugs back in (declared writes)
trigger   what activates it
```

The declaration is the authority. A function has no ambient authority; it
has the reads its selector declares, the writes its sink declares, and the
activations its trigger declares.

Couplings are classified by sink-relation into exactly three roles:
propagator, projection, controller.

### propagator

A coupling whose sink is its own source stream: it advances the system's
state. The agent loop is the privileged propagator: thread stream -> model ->
thread stream, triggered by turn submission.

The word is used in the molecular dynamics sense: the integrator that moves
the whole system forward one step. It is reserved for system evolution and
nothing else.

### projection

A coupling that reads streams and computes derived output discharged
elsewhere: summaries, embeddings, extractions, classifications, indexes.

Law: projections may be lossy; that is their nature. Provenance is mandatory.

This is the event-sourcing, linear-algebra, relational, and cartographic
meaning of the word: a map that drops what it does not keep.

Replaces: observer, derivation, extraction strategy.

### controller

A coupling that discharges into a control stream: feedback that alters future
evolution. Pipeline selection, model profile selection, tool gating,
compaction triggering, conversation routing (which external identity maps to
which thread).

The molecular dynamics ancestors are the thermostat and the metadynamics
bias: functions of the trajectory's history coupled back into the dynamics.
The product-layer word for a controller is "hook."

### chaotic

A coupling property: its output is not reproducible from its inputs.
Model-backed functions are chaotic by definition.

Law: chaotic output must discharge to a stream as events. It can never be a
view, because there is nothing deterministic to regenerate.

### view

A deterministic, regenerable computation over streams. Indexes, graphs, read
models, retrieval caches.

Law: a view can be deleted and recomputed without losing truth. If deleting
it would lose truth, it was an event that got misfiled.

### selector

The declared read scope of a coupling or context source: which streams,
ranges, kinds, and filters.

Law: selectors are declared, not ambient.

### trigger

The activation leg of the coupling: an event-kind predicate plus activation
policy (thresholds, schedules, quotas, depth limits).

Law: trigger names are event kinds. Quiescence is enforced by budget, not
assumed from convergence: trigger quotas and propagation depth limits are
kernel policy, because chaotic couplings have no fixpoint.

Law: every activation chain originates in a witnessed event. The system is
never its own first mover; autonomy in the strict sense — closure of the
trigger graph — is deliberately unconstructible. Standing activation is a
mandate, witnessed when conferred.

### discharge

The act of a coupling appending to its sink stream.

Law: the sink is the dangerous leg of the coupling. Write authority on a
stream is authority over truth; discharges are declared, attributed, and
observed.

### assembly

The deterministic, budgeted projection family that compiles selected events
and views into boundary-visible context, emitting a receipt.

Law: assembly selects and arranges; it never creates. Anything that needs
creating (a summary, an extraction) is a separate projection whose output
discharges first and is assembled later.

Replaces: context strategy (the function half), prompt builder.

## Surfaces

### surface

A faithful re-presentation of a contract on another boundary: operation to
LLM tool, CLI command, HTTP route, MCP export, bash builtin; runtime object
to IO interface. Surfaces move no data forward and make nothing new happen;
they show the same thing somewhere else.

Law: every surface is faithful, or it is illegal.

Replaces: projection in the lossless sense used by earlier Verlet docs. The
README law "projections must be lossless or illegal" becomes "every surface
is faithful, or it is illegal."

## Authority

### operation

The canonical executable contract: describe/call exports, typed input and
output, declared effects, required capabilities. The unit everything surfaces
from.

Operations are runtime-kind agnostic: a wasm guest and a kernel-native
implementation publish the same contract shape through the same gate. A
kernel-native record is synthesized and published only by the kernel itself
and content-addresses its contract; its implementation is the running kernel,
versioned with it.

Law: only the kernel publishes kernel-native records. A user-supplied package
claiming the kernel runtime is rejected at the gate — anything else would let
a publish hijack kernel dispatch.

### principal

A named identity within a tenant, on whose authority effects act: declared
and revoked as witnessed records, carrying a kind from which its authority
class derives. The identity an envelope resolves to and a boundary session
carries. Kinds are few and machine-facing — operator (host authority),
adapter (ingress submission), member (reserved, unimplemented: end users
reach the system through adapters, recorded as adapter testimony). Agents
are not principals; they act under mandate, handle, or remote attribution.
The kernel receives principals; it never provides them — identity services
live above it. Named 2026-07-20 ahead of implementation per the naming law
(kernel ADR 0008).

Law: every connection to a privileged surface resolves to a principal before
it may call any method, and every authorization decision, allow or deny, is
recorded with the principal it was made for.

### credential

The witnessed binding of a bearer secret to exactly one principal. Minting
and revocation are witnessed events naming the authorizing principal; the
record holds only the secret's digest and metadata. Named 2026-07-20 ahead
of implementation per the naming law (kernel ADR 0008); the shape follows
the stream-sync credential authority already in the kernel.

Law: secret material never enters the record, and a credential resolves to
exactly one principal.

### tenant

The isolation unit that owns streams, principals, and published
artifacts; the coordinate a stream already names. One tenant per instance
is the deployment model (standalone daemon or hosted, see [instance](#instance)
and [host](#host)); the wall between tenants is separate instances with
exclusively owned roots, not an in-kernel boundary. Long load-bearing in
code and canon prose; given a headword 2026-07-20 (kernel ADR 0008).

Law: every stream and principal names its owning tenant; nothing
crosses a tenant boundary implicitly.

### tool

A model-visible surface of an operation: direct, bash, or imported protocol
boundary. Tools are surfaces, not implementations.

Law: nothing mutable backs a tool row. A tool definition placed in the
model's static tool set surfaces only an immutable contract — a published
operation or a pinned import. Mutable external universes surface through the
search surface (search, describe, call: contracts arrive in context as
witnessed content, man-page style, never as rows); pinning is the only
passage from universe to row.

Corollary: schema-constrained generation (provider-side grammar
enforcement) is only available to rows, because only an immutable contract
can honestly constrain a grammar. Imported contracts that stay mutable get
call-time validation against the witnessed schema hash plus in-context
learning, fail closed.

### effect class

The retry contract a tool surface declares: pure, idempotent, or
at-most-once. One dimension of the operation's declared effects, carried on
the tool row and witnessed in the bind receipt beside the rest of the
bound surface. An undeclared class is at-most-once: the conservative
reading is the default, and a stronger class is the author's explicit
claim, not the kernel's inference. Named 2026-07-16 ahead of implementation
per the naming law.

Law: recovery honors the effect class. A pure or idempotent invocation
interrupted before its witnessed outcome may re-execute as a new witnessed
attempt; an interrupted at-most-once invocation never silently re-executes —
it reaches its outcome only by reuse of a recorded result or by a witnessed
failure naming the interruption.

### fingerprint

The deterministic identity of one tool invocation's arguments against one
bound snapshot, witnessed on the tool-call request event. Invocation-scoped:
the handle's dispatch identity is a different word for a different scope —
retry of handle-returning calls — and its meaning does not change.
Named 2026-07-16 ahead of implementation per the naming law.

Law: a recorded outcome is reused only under a matching fingerprint within
the same snapshot. A mismatched fingerprint is a different invocation;
reuse across it would ascribe one invocation's outcome to another.

### pin

The acceptance of a witnessed external contract as a published,
content-addressed operation record. A pinned import depends on nothing
external: the remote endpoint becomes mere placement, and calls fail closed
against the pinned contract rather than tracking the live one.

Law: a pin is a publish. It passes the same gate, carries the same receipt,
and resolves through the same alias discipline as any published operation.

A machine-readable contract corpus (an OpenAPI document) pins as a batch:
the publisher compiles the witnessed document into operation records through
the same gate, one receipt per record. The compiler is surface grammar
(`.import.toml`); the kernel sees only pins.

### package

The authoring and publishing unit (`verlet.tool.toml`): declares operations,
runtime source, fixtures, capabilities, and surfaces.

### artifact

Immutable published bytes, content-addressed.

### bundle

A hierarchical content address: a tree of name→component entries in which
every component is named by its hash — opaque bytes or another bundle — and
the bundle's own address is the hash of the tree. The general form of
composite published content. Distinct from `artifact` (flat immutable bytes;
a bundle's canonical tree encoding is published as one) and from `package`
(the authoring and publishing unit, which may publish bundles). Components
dedupe across bundles by construction, and a consumer may materialize any
subset of components without holding the rest.

Named 2026-07-13 ahead of implementation per the naming law (skill-bundle
redesign): the skill word stays Public Jargon — a skill is surface
convention that lowers at bind to bundle components as text, exec, and blob,
and hooks never lower (standing authority must not arrive by content
ingestion).

Law: a bundle addresses its components individually. A receipt that names a
bundle can therefore name the exact component bytes that acted —
per-component witness is what the tree buys, and an encoding that cannot
yield it is not a bundle.

### register

The operation that admits content into a scope the runtime provisions: the
acceptance of a bundle or artifact into a registry ahead of materialization
into a provisioned scope (sandbox, serverless). The skill-surface `publish`
verb narrows into it. Named 2026-07-13 ahead of implementation per the
naming law.

Law: register exists only where the provisioner is the runtime. Where the
user provisions the scope (host mode), acquisition is the user's — via
whatever ecosystem tooling they choose — and the runtime's only verb is
witness: hash what the scope holds into the bind receipt. A register
operation over user-provisioned scope would manage state the runtime does
not own, inheriting blame without control.

### binding

The scoped availability of a published operation to a tenant, project, or
thread boundary. A binding begins with a witnessed [attach](#attach) event
and ends with a witnessed [detach](#detach) event; a thread's current
toolset is the fold of its attach and detach events, and every catalog or
router over that toolset is a derived view — deletable, recomputable, never
authoritative. The binding's configuration (allowed secrets, allowed
private-network origins and methods, future per-tool settings) rides on the
attach event; in code the field is `attachment_config`.

Law: hot-swappable between turns; a running turn keeps the toolset it
started with. Attach and detach take effect at the next turn boundary.

Law: an action receipt cites the attach event that made its tool available.
Every tool-call receipt chains back to the authority that admitted the tool.

Law: resume re-folds the thread's own stream. There is no standing document
whose hash can mismatch the thread's history.

(Anchor word pick 2026-08-11: binding is the surviving concept word;
attach/detach are its event kinds. "Attachment" as a noun is retired from
canon prose; the code-level `attachment_config` field name stands.)

### attach

The witnessed event that starts a [binding](#binding): the content-addressed
operation ref, the binding configuration, who requested it, and which
journaled policy decision authorized it when one was required.

Law: every attach is journaled with requester, decider, and configuration,
and the configured policy is consulted before the append. Who may attach is
policy, never law; the compiled invariant is that the record always answers
who asked and who said yes.

### detach

The witnessed event that ends a [binding](#binding). A scheduled detach — a
detach with a future effective time, executed by the runtime when due —
replaces every use the deleted expiry machinery ever had. A detach of a
tool mid-call lets the running turn finish; it takes effect at the turn
boundary like any other.

### alias

A mutable name. Aliases must resolve to immutable records at publish or run
time and produce a receipt.

### anchor

The external authority a publish gate validates against: tenant policy, or a
human whose decisions enter as witnessed events. The anchor lies outside
the component it grounds; the regress of authority terminates there.

Law: the anchor is never modeled. No manifest, schema, or coupling takes the
anchor as its object — a system that predicts its grantor's approvals has
moved the anchor inside and is granting itself powers with extra steps.

### petition

A request to widen a thread's authority, carrying provenance to the need
that motivated it, decided outside the component that made it.

Law: the approval topology is a DAG anchored outside the component: a
connected component of agents cannot mint new authority for itself.

(The original publish-gate mechanism was never built; `GrantPetitioned` is
frozen dead vocabulary. The authority redesign narrows petitions to
authority-widening attach requests — the authority attach/detach design note, August 2026.)

### condensation

A discharged proposal to settle recurring interpretive work into a published
operation: the package distilled from repetition, carrying provenance to the
witnessed episodes it generalizes, consumed by the publish gate. The
repetition-driven path to the gate; the petition is the denial-driven one.

Law: a condensation confers no new capability. The condensed operation's
required capabilities are bounded by the authority its episodes already
held; any excess is a petition, and the two are never bundled.

Law: a condensation names its episodes. Fixtures that replay no witnessed
episode pin the operation to the proposer's imagination, not to the world.

### demotion

The receipted narrowing of a binding or alias on accumulated failure
evidence: re-bind to a prior record, unbind, or return the work to
interpretive execution. The narrowing dual of the petition path: a petition
widens the universe through the gate; a demotion shrinks what is bound from
it, at the bind layer.

Law: a demotion carries provenance to the evidence that motivated it and
takes effect at next bind; a running turn keeps its snapshot. Demotion
narrows form, not authority: revoking authority or a mandate is a separate
act with its own receipt.

### degradation

A model-visible, witnessed capability omission at bind: content was bound
whose component the lane cannot honor (a script where no OS exists to run
it), so the bound surface is narrower than the declared content. Not a
`demotion` — a demotion narrows a binding after the fact on accumulated
failure evidence; a degradation is imposed by the lane at bind time, before
any failure, and the content itself is unchanged. Named 2026-07-13 ahead of
implementation per the naming law (skill-bundle redesign, serverless lane).

Law: a degradation is witnessed in the bind receipt AND stated in the
model-visible material assembled from the binding. The receipt alone is
insufficient: an index that presents an unrunnable component as available is
an unfaithful surface, and leaving the model to discover the omission by
burning a turn is the failure mode the word exists to forbid.

### mandate

A standing activation authority: grantor, scope, budget, expiry. The admissible
form of long-horizon purpose — leased, never owned. Operationally
indistinguishable from autonomy except at revocation time, and revocation
time is the point.

Law: every mandate names its grantor, budget, and expiry. No mandate renews
itself.

Replaces: mission (banned).

## Composition

### manifest

The versioned declaration of what an agent or policy is allowed to be. For
an agent: identity, model profiles, tools, resources, context pipelines,
policies, runtime defaults. For a policy: budget, spawn roster, io schema —
the meta block of its authoring surface. The manifest declares the
universe; the bound policy selects within it.

DEPRECATED 2026-08-10 (anchor-directed): the manifest as standing authority
document is replaced by a [preset](#preset) expanding into an
[opening sequence](#opening-sequence) of attach events (design settled
2026-08-11, the authority attach/detach design note, August 2026). This entry describes
the current code until the migration slices land.

### preset

The stored, versioned convenience document that expands into an
[opening sequence](#opening-sequence) at thread start. Replaces "manifest"
as the user-facing artifact. Content-addressed like everything else, with
aliases for human names. The daemon's default agent becomes the default
preset, synthesized from daemon config exactly as today, expanded instead
of bound.

Law: a preset confers nothing by existing. Only its expansion into
witnessed attach events makes tools available; authority lives in the
events, never in the document.

### opening sequence

The attach events appended between thread start and the first turn. This is
what "binding a manifest" becomes: not a standing authority document, just
the recorded first attachments, folded like any later ones.

### default manifest

The manifest the daemon synthesizes from its own configured envelope and
publishes at startup, like any other manifest. A thread start that names no
manifest binds the default manifest: the ambient envelope is a declaration,
not interpreter state.

Law: every thread binds a manifest; a start that names none binds the
default manifest. The default manifest is republished only when its content
changes, and nothing about it is special at bind time.

Same deprecation note as manifest: describes current code; becomes the
default [preset](#preset) expanding to an
[opening sequence](#opening-sequence) under the redesign.

### policy

The artifact bound at the admission boundary of a thread: given the folded
state of the thread's stream, it computes the admissible continuation set.
Two modes:

- **strict** — a deterministic continuation function. Only moves it names
  are admissible, and the kernel enforces that; replay of the journal
  re-derives every decision. The surface word for policy code in this mode
  is "workflow" (Public Jargon).
- **adaptive** — chaotic, model-backed. The model selects the continuation
  inside the authority and budget envelope the manifest declares. The privileged
  propagator's default agent loop is the degenerate adaptive policy.

A policy is content-addressed and bound like any published artifact
(`policy.bound` names the hash); rebinding is a new hash, and in-flight
threads keep their snapshot. A hosted policy lowers to a published operation
whose required capabilities are the admission verbs.

Law: the policy selects within the universe its manifest declares; it never
widens it. Widening is a petition.

### admission

The kernel's continuation decision at a thread's boundary: which move runs
next, out of the **admissible** set the bound policy computed. Admission is
recorded (`admission.decided`), making the decision — not just the outcome —
part of history.

Law: every continuation the kernel executes was admitted. There is no
side door around the admission boundary, for either mode.

### envelope

The witnessed unit of boundary ingress: one external delivery, translated at
the boundary into the system's terms and consumed by admission. Carries
external provenance (which system, which external identity, which delivery),
the principal and tenant it resolves to, and a dedupe identity. Already
load-bearing in the ingress contracts and in `claim`'s law; named 2026-07-18
ahead of the managed-lane adapter contract per the naming law. Principal
attribution names its resolution path via a scheme: `route:`, `mandate:`,
`handle:`, `remote:` (kernel ADR 0007), and `caller:` for submissions by a
boundary-authenticated session (kernel ADR 0008, added 2026-07-20).

Law: an envelope names its external delivery and resolves to a principal
before admission, and redelivery dedupes on its delivery identity. An
arrival the runtime cannot attribute to a delivery and a principal is not
admissible as witnessed ingress.

### claim

The witnessed acceptance of sole responsibility for an admitted ingress
envelope's outcome, appended expected-tail fenced to the resolved thread's
control stream before any non-idempotent effect. Named 2026-07-10 ahead of
implementation per the naming law; the event kinds it introduces
(`io.ingress.claimed`, `io.ingress.settled`) are ratification-gated and
pending on the docket (ADR 0003 in the kernel repo).

Law: at most one claim exists per ingress envelope. A claim names its
intended outcome and carries provenance to the ingress witness and admission
decision it executes; no non-idempotent admitted effect precedes its claim.

### settle

The witnessed terminal outcome of a claim, carrying provenance to the claim
and to its execution evidence, and a discriminator for whether execution or
recovery produced it.

Law: every claim settles exactly once, by execution or by recovery. A settled
claim is terminal: redelivery dedupes against it and repeats no effect.

### hold

The access one tool invocation takes on a keyed runtime resource for its
duration: shared or exclusive. Holds are derived by the kernel from the
bound tool's surface, never supplied by the model, and are witnessed on the
tool-call request event. Two invocations of one batch may run concurrently
only when no key is held exclusively by one while held at all by the other;
an invocation with no derivable holds takes the global key exclusively.
Named 2026-07-13 ahead of implementation per the naming law (chosen over
overloading "claim", which is ingress-scoped).

Law: a tool invocation executes only under its witnessed holds, and results
enter history in call order regardless of finish order. Schedule legality
is decidable from the record alone.

### pipeline

A named, declared set of context sources in the manifest. Ordered list; first
is the default. Selecting among declared pipelines is runtime policy and the
future controller point, mirroring model profiles.

### source

One leg of a pipeline: an assembler ref, a selector, and a budget share.
Sources are independent of each other; the kernel performs the deterministic
merge, budget fit, and receipt.

### agent

A runtime object: a manifest, a thread, and the privileged propagator wiring
between them. An agent is not a prompt.

### thread and turn

A thread is the durable execution scope that owns a primary stream. A turn is
one trigger-to-quiescence cycle of the privileged propagator.

### handle

A durable reference to work in flight, returned by a call in place of its
value: a spawned thread, a host process, any operation whose execution
outlives the turn that invoked it. The handle names what was started; the
stream carries what becomes of it. Thread ids and process ids are the two
current instances; their verb families (spawn/submit/wait/status/cancel,
exec/write/poll/terminate) are per-kind surfaces over this one primitive.

Law: a handle-returning call declares a dispatch identity and is idempotent
on it — a retried call returns the original handle, never a second
execution. Every handle reaches exactly one witnessed terminal outcome
carrying provenance to the originating call; awaiting a handle yields that
outcome and nothing else, and consumers dedupe against it.

### workspace

The filesystem scope a thread executes against: one mutable file tree per
thread, rooted at the thread's resolved working directory.

Law: one workspace per thread. Every filesystem surface within a thread —
virtual bash, operation invocation, resource mounts — re-presents the same
workspace; two surfaces showing different trees under the same scope means at
least one is unfaithful. Visibility and mutation are declared, not ambient;
filesystem scopes name paths within this one tree.

### harness

The emergent name for an agent's resolved execution envelope. A read model
that can be exported, diffed, and inspected. Never a declared field.

### kernel

The privileged core that compiles the layer-0 laws and mediates all
authority: stream order, event immutability, origin and provenance,
attachment and mediation, budgets, receipts. Mechanism, never policy — everything
above it is named, versioned, and swappable ([the Ladder](laws.md)). The word
is used in the operating-system microkernel sense: not an evaluator, not an
orchestration library.

Law: the kernel owns exactly what could make a receipt lie. Anything whose
change only alters future behavior, receipted, lives above it.

### runtime

The kernel in motion: the running instance that executes turns and attests
facts. When a witnessed event's authority is "the runtime did it," the
runtime's attestation is that authority. Orthogonal to placement: where the
runtime runs changes nothing about what it may attest.

Law: runtime authority is attestation. Witnessed events and receipts record
what the runtime did; nothing else speaks with the runtime's voice, and the
runtime never authors interpretation without provenance.

### instance

One embedded kernel: a single tenant's runtime constructed as a library
value inside a hosting process — its stores, background tasks, identity
authority, and lifecycle. An instance owns its filesystem roots exclusively
and its shutdown is explicit; nothing tears an instance down as a side
effect of being dropped.

Law: instance shutdown is a hosting event, not a thread-lifecycle event.
Thread lifecycle records stay nonterminal across it; a rehomed instance
recovers its threads from the journal.

### host

The process that runs many instances behind one boundary listener and routes
each authenticated connection to exactly one of them. Routing context — the
credential the connection presented — selects the instance; the instance's
own identity authority then decides. The host is plumbing between the
boundary and the instances, not a new seat of authority.

Law: the host never speaks with an instance's authority. Routing is
selection, not authentication: a credential the selected instance rejects is
rejected outright, and no other instance is probed. Identifiers in a request
body never override the connection's routed instance.

### conductor

An external client driving a thread through the boundary surface: submitting
turns, awaiting admissions, joining spawned threads — the SDK's relationship
to the kernel. Named so the canonical coupling role `controller` stays
unambiguous: a controller is a coupling discharging into a control stream; a
conductor is outside the system, and everything it does arrives as witnessed
ingress through granted surface calls.

Law: a conductor holds no ambient authority. Its calls are granted,
witnessed, and indistinguishable in the journal from any other boundary
ingress.

### orchestrator

The distinguished standing conductor: the deployment-plane component that
supervises a fleet of runtimes and decides when and why agents run. It
hosts the adapters, runs the scheduler that injects witnessed envelopes for
due time-based mandates, writes the placement bindings that pair each agent
with a runtime instance, and invokes published operations, agents, and
policies by their registry ids. Everything a conductor is, an orchestrator
is; the word marks the standing component, not a new kind of authority.

Law: the orchestrator is a client of the record, never a second store. Its
own state — schedule mandates, placement bindings, fleet membership, run
outcomes — is witnessed events on streams under its own principal. An
orchestrator holding orchestration truth in a private database is
malformed: the record of why an agent ran must not be able to drift from
what happened.

Law: a placement binding written by the orchestrator carries a lease epoch.
A runtime presents its epoch when appending to a placed agent's journal;
a stale epoch fails closed. One writer per journal is fenced, never
assumed.

### guarantee tier

What an execution target can prove about its own runs. `attested`: a
runtime whose turns are journaled, admitted, and receipted — the runtime's
attestation carries authority. `recorded`: a foreign harness driven
through a shim; the orchestration around the run (invocation, input,
outcome, timing) is witnessed, the run's interior is not, and no resume is
promised.

Law: the tier is declared at the invocation surface and recorded with the
run. No consumer may treat a recorded run as attested.

### client stream

A stream owned by a boundary client rather than by a thread lifecycle: the
store lane through which an orchestrator (or any principal the boundary
admits) keeps its own state in the record. Records in a client stream are
witnessed — they enter from outside the runtime — and carry the writing
principal. The stream id wears the `client:` prefix so every sweep that
walks thread or control streams passes it by.

Law: a client stream never schedules work. Appending to one is a statement
of fact, not a petition; nothing admits, queues, or wakes because of it.
A record whose append is meant to cause execution belongs on an ingress
surface, where admission can refuse it.

Law: client records declare their payload schema cohort, and the kernel's
own cohort is closed to them. The kernel records the declaration; it never
interprets the payload.

### placement

Where execution physically runs. Orthogonal to authority: moving placement
changes nothing about authority, streams, or receipts.

Law: placement attaches at the binding — or the conductor boundary call that
creates one — never inline in a model-visible tool call. The model names what
runs and under which contract; the deployment decides where. A placement
lever on a tool surface would hand the model deployment authority and bake
endpoints into the transcript, so replay across deployments would carry
stale infrastructure as if it were history.

### backing

The substrate behind a guest-visible mount: which store or source supplies
the bytes under one fixed guest path. The storage dual of `placement` (where
execution runs); backing is where a mount's content lives. Selected at the
binding plane and witnessed in the bind receipt; the guest surface is
uniform across backings. Named 2026-07-13 after three independent arrivals
of the same shape: spill (session / host / remote), workspace (EMO-433 host
mount), and skills — the third arrival later dissolved the same day into
mount-free workspace discovery (no skill mount exists in host mode, so
nothing there carries a backing), leaving spill and workspace as the
standing instances.

Law: backing attaches at the binding, never inline in a model-visible
surface, and the bind receipt witnesses the selection. Changing a mount's
backing must not change the guest contract — a backing that leaks through
the surface (paths that only exist under one backing, semantics that differ
by substrate) is an unfaithful surface. Where a backing cannot honor part of
the contract, that is a degradation and carries that word's law.

## Conformance

Named 2026-07-06 (anchor-ratified), ahead of implementation per the naming
law. These words exist because the laws are being published in checkable
form (the unified-agent-semantics paper, §10.1); this section names the
artifacts that check them.

### criterion

An observable behavioral contract lowered from the laws: a claim a running
system either exhibits or does not, decidable from the system's record and
declared interfaces alone. The paper's C1–C13 is the current normative set.

Law: a claim whose deciding requires reading component code, operator
testimony, or trust in instrumentation is not a criterion. The set changes
only through the loop: adding a criterion is conservative extension;
changing one's meaning is case 2.

### battery

The executable form of the criteria: one test per criterion, run against a
candidate system's records and declared interfaces, emitting a scorecard.
The battery is the oracle pointed outward — the same mechanical-check
discipline the loop applies to the canon, applied to systems claiming the
abstraction.

Law: the battery implements the criteria and nothing else. A test with no
criterion is malformed; a criterion with no test is an open ratchet item.
The test is the guard, so every amendment to the criteria lands with its
test in the same change.

## Simulation

Named 2026-07-11 (case 1, ahead of implementation per the naming law). The
battery is the oracle pointed outward, at systems claiming the abstraction;
these words are the oracle pointed inward, at the kernel's own interleavings.
The artifacts live in the kernel repo's deterministic simulation harness
(ADR 0004); the words live here because they bind.

### fault plan

The deterministic expansion of a seed into a schedule of faults over the
harness's named operations and cuts: store failures before or after append,
duplicate leases, provider failures, process death at a named cut.

Law: a fault plan is a pure function of its seed and a versioned fault
vocabulary. Same seed, same schedule; a vocabulary change is a new version,
never a reinterpretation of old seeds.

### scenario

One seeded, bounded run of the simulation harness: an operation sequence and
a fault plan derived from the same seed, executed with every declared
invariant checked after every step.

Law: a scenario is reproducible from its seed and harness version alone. A
failing scenario's receipt is its seed plus the normalized transcript; the
minimized failing sequence joins the fixed regression corpus, so every
defect the search finds retires into a deterministic guard.

## Banned And Deprecated Words

| banned | replacement | why |
| --- | --- | --- |
| projection (for lossless re-presentation) | surface | every source domain (linear algebra, relational, cartography, event sourcing) uses projection for lossy maps; fighting that prior forever is friction |
| propagator (for any coupled function) | coupling; then propagator, projection, or controller by sink-relation | propagator is the integrator, not the analysis or the thermostat |
| observer | projection or controller, by sink | "observer" hid the sink and trigger legs |
| derivation | projection | one word for the lossy function |
| context strategy | pipeline (declaration) + assembly (function) | "strategy" mixed config with execution |
| ObservationStore (as truth) | discharged events on streams; retrieval indexes are views | one truth category |
| ContextStore | receipts (events) + view caches | same |
| capsule | package, artifact, or operation, by intended meaning | three meanings hid under one word |
| memory | product word; lowers to projections + pipelines over streams | not a kernel primitive |
| hook | product word for controller | needs no kernel primitive of its own |
| subagent | product word; lowers to topology/delegation over manifests and threads | same |
| harness (as manifest field) | emergent read model only | settled in the manifest audit |
| mission | mandate | manifest-resident telos closes the trigger graph; purpose is leased (grantor, budget, expiry), never owned |
| ceiling (for the authority terminus) | anchor | a DAG cannot ground at a ceiling; the immovable belongs low in this house's imagery, and the trust-anchor prior (authority assumed, not derived) is the concept verbatim |

## Public Jargon

Layer-3 words exist so users recognize what they are buying. The public
dictionary translates without leaking upward:

```text
"subagents"   -> delegation topology: child manifests, child threads
"memory"      -> projections discharging to durable streams + pipeline sources
"hooks"       -> controllers on lifecycle event kinds
"skills"      -> witnessed workspace discovery (bind-time traversal + index),
                 or records compiled at import (blob/context/operation);
                 hook-shaped components never lower
"sandbox"     -> placement + attachment config
"workflow"    -> policy code composing operations and agents via spawn/join
"connectors" / "adapters"
              -> conductors translating external deliveries into envelopes,
                 through the ingress surface; deployment-plane, never a
                 kernel primitive
"schedules"   -> a mandate whose activation policy is time-based; the
                 orchestrator's scheduler injects witnessed envelopes when due
"triggers"    -> product word for schedules (above); never the canonical
                 `trigger` (a coupling's activation leg) — the product word
                 lowers to a time-based mandate under the owner's authority
"COOL"        -> the battery ("Canonical Observable Operational Laws")
"certified cool" / "COOL-13" -> a passing scorecard against the battery
```

Public docs may say "supports subagents, memory, and hooks." Kernel code,
manifest schemas, and event kinds never do.

The project and runtime's public name is **Verlet** (anchor-decided
2026-08-03, superseding the 2026-07-06 ratification of "Cooldis";
verlet.dev is the public page and cooldis.com redirects to it). The
paper's NAME-TBD placeholder resolves to Verlet at ship; "COOLDIS" and
"certified cool" survive in the paper as a deliberately unelaborated
footnote (anchor disposition 2026-08-03). The badge grammar's relation
to the old name remains deliberately undocumented in public artifacts:
found, never explained.

Surface grammar (authoring-file suffixes, compiled by the publisher — the
kernel never parses them): `.tool.toml`, `.agent.md`, `.workflow.ts`,
`.ensemble.toml`, `.import.toml` (declares a witnessed external contract
corpus and the subset to pin). The suffix is the public word; what lands
after compile is canon words only (operation, manifest, policy).

## Ancestry

The CS ancestry of these words — partial evaluation and the Futamura tower,
traits and structural composition, algebraic effects, initial encodings,
template metaprogramming — and the precise departures from each are documented
in [The CS Ontology](grounding/cs-ontology.md) (grounding tier). The physics
and philosophy ancestry
lives with the epistemic model. How this lexicon changes is
[README.md](README.md), How It Changes: wrong content is struck directly;
contested changes are plain questions for the anchor.

## Open Naming Decisions

- Scope of the `capsule` retirement: rename in docs first; code identifiers
  migrate opportunistically during the WS-D cleanup thread.
- Whether public docs say "observation" or "discharged event"; the kernel
  treats them as the same thing.
- The friendly public name for the privileged propagator ("the agent loop").

Resolved decisions leave this list: the chaotic provider-side-state category
is named (`residue`, 2026-07-16, subsuming the D4 interim ruling as its
unreadable mode), and the handle terminal-outcome question closed as docket
D-12 (2026-07-12, settle stays ingress-scoped — see amendments).
