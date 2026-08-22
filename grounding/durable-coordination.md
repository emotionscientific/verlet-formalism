# Durable Coordination And Resume

> **Dating note.** Written July 2026, before the August 2026 authority recast.
> Where this document says *grant*, the current word is *binding* (an
> operation is attached to a thread by a recorded attach event and absent
> until then); who may attach is policy, not law. The laws and lexicon in
> this repository are current; this document is background and will be
> recast in place.

Status: grounding design note. This document translates the formalism's stream,
event, coupling, receipt, binding, and placement vocabulary into the operational
durability model needed for SDK bindings, async task queues, workflow-shaped
coordination, restart/resume, and deployment rollover. It defers to
[the formalism](../formalism/README.md) and restates canon in implementation
language.

## Thesis

Durability is not a process feature. Durability is an event-stream feature.

The useful claim is not "retry makes agents reliable" or "deterministic replay
solves agent execution." The useful claim is narrower and stronger:

```text
The runtime does not forget the coordination truth.
Already witnessed or discharged facts are not silently re-run.
A new worker can decide the next lawful step from durable records alone.
```

That gives Verlet the good part of workflow systems without making process
memory, task-queue names, SDK heap variables, or provider callbacks into the
source of truth.

## The Three Ledgers

Verlet durability has three layers:

```text
publish ledger
  content-addressed operations, manifests, couplings, schemas, packages,
  fixtures, aliases, and HEAD pointers

runtime event streams
  append-only typed facts at every ABI boundary: activations, tool calls,
  model calls, process exits, approvals, provider results, terminal states,
  receipts

materialized state
  read models for scheduling, loaded handles, UI, waiting queues, indexes,
  cache locality, and debug surfaces
```

The publish ledger says what a run was allowed to mean. Runtime streams say what
actually happened. Materialized state helps the system move quickly, but must be
reconstructible from the ledgers or safely disposable.

Mutable names such as `HEAD`, aliases, defaults, and tenant bindings may move,
but a running activation binds immutable records. A new bind may use a new
record. An old run keeps the hashes it already receipted unless a declared
migration or plan patch is accepted at a boundary.

The V1 record envelope and storage adapter boundary for these streams is
[ADR 0001: Stream Schema V1](https://github.com/emotionscientific/verlet-kernel/blob/main/docs/adr/0001-stream-schema-v1.md). This
document explains why the model exists; the ADR pins the schema contract the V1
release candidate must implement and validate.

## Boundary Facts

Every contract boundary should emit typed events automatically. Developers do
not "remember to log" resumable truth; the ABI wrapper records it because the
boundary cannot be crossed without the kernel.

Examples:

```text
activation.enqueued
activation.claimed
activation.started
activation.completed
activation.failed
activation.cancelled
activation.timed_out
activation.suspended

operation.invocation.requested
operation.invocation.completed
operation.invocation.failed

thread.spawned
turn.submitted
turn.completed
turn.incomplete

model.call.started
model.call.completed
provider.call.failed

process.started
process.exited
process.terminated_by_signal

approval.requested
approval.resolved

plan.compiled
plan.patched
deadletter.recorded
```

The exact names belong in the event-kind vocabulary, but the shape is the
point: if a fact can change the next activation, it must either be deterministic
from prior events or enter a stream as a witnessed or discharged event.

Absence is not a resume protocol. If a process receives SIGTERM, a tool call
times out, a model stream ends without a final turn, or a worker loses its
lease, the runtime records an explicit terminal, incomplete, timed-out,
suspended, or abandoned fact. A resumed worker infers continuation from those
facts, not from guessing what a missing log line meant.

## Activation Identity

Do not overload provenance as a lock. Provenance explains why an event exists;
leases fence who may advance a live activation.

Use separate coordinates:

```text
activation_id   durable unit of work
attempt_id      one execution attempt for an activation
lease_id        current claim by a worker or placement
fencing_token   monotonic token accepted by the store
event_id        immutable fact
provenance      source events, function ref, version, config hash, receipts
```

A stale worker may still wake up and try to write. The store rejects terminal
events whose lease or fencing token is no longer current. The accepted terminal
event still carries provenance, but provenance is not authority.

## Retry Is A Controller Decision

Durability preserves coordination truth. It does not make retry correct.

Agent failures have different shapes:

| Failure shape | Default controller behavior |
| --- | --- |
| Code bug | Do not blind-retry. Record failure evidence, route to repair or publish a fixed operation/plan. |
| LLM mistake | Do not blind-retry. Surface the failure to the agent as context and let it revise the plan. |
| Provider outage or quota | Let the model/provider router handle redundancy, budgets, and fallback. Record the provider evidence. |
| Remote resource or network hiccup | Retry only if the operation contract says the effect is idempotent or reconcilable; otherwise surface ambiguity. |

Operation contracts should declare effect class, idempotency key behavior,
retry eligibility, timeout semantics, and reconciliation behavior. If a
side-effectful call is ambiguous, reconcile before retrying. For money-moving,
email-sending, ticket-writing, or device-control operations, "durable retry" is
not enough; the domain state machine and external idempotency contract are part
of the operation contract.

The retry controller consumes failure events and emits one of:

```text
activation.retry_scheduled
failure.reported_to_agent
provider.route_changed
deadletter.recorded
approval.requested
plan.patch_requested
```

This keeps retry as declared policy over stream facts, not a hidden loop inside
an SDK runtime.

## SDK Bindings

Language SDKs may look imperative, but they must not own the continuation.

An SDK `await` is lawful only when the awaited thing has a durable activation
id. The SDK can be a compiler, trampoline, or projection over the runtime:

```text
TS/Python/Rust/Bash/agent turn
  emits commands or declarations
  lowers to activations and receipts
  waits on event-stream facts
```

The SDK may hold temporary local variables while the process is alive, but
restart semantics come from the activation ledger. If a JS closure, Python
object graph, shell script variable, or parent-agent context is required to
continue, the feature is not first-class Verlet durability yet.

## Workflows And Coordination Runs

"Workflow" is product language. Kernel words are enough:

```text
thread          durable agent execution scope
turn            one trigger-to-quiescence cycle
operation       executable contract
coupling        declared coordination over stream facts
activation      durable unit of scheduled work
receipt         runtime act recorded as an event
```

`activation` is used here in the operational sense already present in the
coupling scheduler design: one durable occurrence of scheduled work caused by a
trigger or command. If it becomes a public ABI object rather than implementation
language, it must be named in the lexicon first.

A workflow-shaped run is a coordination pattern over those words. It may enqueue
operations, spawn child threads, wait on completions, fan out, fan in, retry,
deadletter, ask for approval, and patch the plan. Its middle layer must be an
event stream, not a script runtime.

Dynamic workflow therefore means:

```text
chaotic agent proposes or revises a plan
-> plan or patch is recorded with provenance
-> deterministic scheduler advances activations from stream facts
-> chaotic outputs re-enter as events before they affect continuation
```

This is compatible with Temporal-style durable orchestration, but differs in
the important place: agent turns are visible stream events, not opaque
activities whose internal decisions disappear behind a task result.

## Resume And Rollover

A worker may continue a run only from durable facts under the bound contracts:

1. Resolve the thread/run identity and the immutable manifest, operation,
   coupling, schema, and package refs already bound for the current boundary.
2. Read terminal, waiting, suspended, leased, and deadletter facts from the
   relevant streams.
3. Rebuild materialized state such as pending activations and waiting approvals.
4. Acquire a fresh lease or fencing token for any activation it will advance.
5. Append explicit continuation, retry, failure, or operator-attention facts.

Zero-downtime update follows the same rule. New code can publish new records and
move `HEAD` for future binds. Old runs do not replay against whatever code
happens to be deployed today; they continue from the content-addressed records
they already bound, or they accept a declared migration/plan patch at a
turn/activation boundary.

This avoids the long-lived workflow trap where waiting executions pin old
process code by accident. In Verlet, waiting executions pin records on purpose.

## Formalism Connection

| Operational term | Formalism lowering |
| --- | --- |
| log | stream of events |
| checkpoint | explicit event or receipt sufficient to reconstruct a boundary |
| retry | controller coupling over failure events |
| schedule | controller coupling over mandate events |
| workflow | product pattern over threads, operations, couplings, activations, and streams |
| SDK function call | surface or compiler over operation/activation commands |
| worker claim | lease/fencing token, not provenance |
| task queue | materialized view plus activation events |
| callback | completion event plus controller coupling |
| dynamic plan | versioned declaration or plan-patch event with provenance |
| process state | materialized state; never truth unless recorded as events |

The practical law:

```text
Durability comes from typed boundary facts, not from replaying process memory.
```

The resume law:

```text
A new runtime may continue only from the latest valid terminal, waiting,
suspended, deadletter, or leased activation facts under the bound
content-addressed contracts.
```
