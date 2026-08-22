# Governed Self-Extension

> **Dating note.** Written July 2026, before the August 2026 authority recast.
> Where this document says *grant*, the current word is *binding* (an
> operation is attached to a thread by a recorded attach event and absent
> until then); who may attach is policy, not law. The laws and lexicon in
> this repository are current; this document is background and will be
> recast in place.

Status: design note, grounding tier — it coheres with
[the formalism](../formalism/README.md) and never overrides it (the third
Futamura projection with a chaotic generator — see
[The CS Ontology](cs-ontology.md)). Vocabulary defers to
[the lexicon](../formalism/lexicon.md); the development process that produced
this note is [the loop](../formalism/loop.md). Decisions here were made and
anchor-approved 2026-06-10; the condensation and demotion paths were named
2026-06-11 (see the [amendments log](../formalism/amendments.md)).

An agent that can widen its own universe is not a new primitive. It is the
existing words — operation, grant, controller, receipt, alias, binding,
budget — wired in a specific shape, plus three laws that keep the shape from
eating its owner.

## The Petition Path

Self-extension is never a runtime act. It is a publish act, reached through
a path on which every leg is receipted:

```text
1  out-of-scope attempt   denied THIS turn; the denial discharges as an event
2  petition               a controller on the denial kind composes a manifest
                          amendment and invokes the publish gate; provenance
                          points at the denial
3  the gate               validates against the anchor; accepts or refuses;
                          publish receipt either way
4  alias moves            new immutable manifest record; old record is history
5  next bind              the widened universe takes effect; the running turn
                          kept its snapshot; the retry now succeeds in-universe
```

The capability that enables step 2 is not a field and not sudo: it is the
publish operation surfaced as a tool, with a grant scoped to `agent://self`.
It rides the ordinary grant machinery, so "granted is not unobserved" comes
free. Without the grant, step 1 still happens and the path ends there.

Fail-closed survives because the denial is real: an attempt never auto-widens
the universe mid-turn. A denial that converts itself into a grant is not a
denial, and a universe that grows on demand was never declared.

## The Condensation Path

The petition widens the universe when it denies; condensation settles it
when it repeats. Same gate, different motive, every leg receipted:

```text
1  recurrence observed   a projection over thread streams discharges a
                         recurrence observation: this task family keeps
                         routing through chaotic work and its chaotic
                         share is not falling
2  condensation          a controller on the recurrence kind composes a
                         package — operation, fixtures replaying the
                         witnessed episodes, resources — and invokes the
                         publish gate; provenance points at the episodes
3  the gate              validates capability-boundedness mechanically:
                         the condensed operation's required capabilities
                         must sit inside the grants its episodes already
                         held. excess capability is a petition, never
                         bundled
4  alias moves           the operation publishes like any other; old
                         records are history
5  next bind             the settled work routes through the operation;
                         the interpretive path remains as fallback
```

The recurrence projection is a view over receipts — the chaotic share of
work per task family — so the trigger input is regenerable and the threshold
is policy. Because a lawful condensation confers no new capability, it is
the natural auto-approve tier of the dial: the human approves powers;
settled form inside existing powers can flow on policy.

Tenant policy may additionally require that any publish's fixtures replay
witnessed episodes, not only condensations. The universal law-form of that
requirement is held pending the anchor (see the
[amendments log](../formalism/amendments.md)).

## The Demotion Path

The dial needs a way down. Demotion is the narrowing dual of the petition:
a denial motivates widening; accumulated failure evidence motivates
shrinking what is bound:

```text
1  evidence accumulates  failure receipts on a bound operation: fixtures
                         failing, calls erroring, drift against the pinned
                         contract
2  demotion              a controller on the failure kinds discharges a
                         demotion: re-bind to a prior record, unbind, or
                         return the task family to interpretive execution;
                         provenance points at the evidence
3  next bind             the narrowing takes effect; the running turn
                         keeps its snapshot; re-condensation may follow
                         later, from fresher episodes
```

Demotion narrows form, not authority — revoking grants or mandates is a
separate act with its own receipt. The ancestry is JIT deoptimization:
residuals carry guards, and a guard failure returns the work to the
interpreter instead of letting a stale specialization rot in place.

## The Anchor

The gate's law is external. Publish validation checks the petition against
authority that lives outside the petitioning agent's manifest — tenant
policy, or a human whose decisions enter as witnessed events. Without an
external anchor, "the model cannot grant itself new powers" is circular: an
agent could amend its own amendment grant and root itself.

The anchor is never modeled (lexicon law). The system holds no schema for
its grantor; a petition selected by predicted approval has moved the anchor
inside. This omission is load-bearing, not modest: a system that contains
its own grounding authority can justify anything.

## Anchoring

Agents may approve other agents' petitions. The approver is its own declared
universe — orthogonal by default, connected at exactly two granted points:
its selector reads the petitioner's petition events, its sink discharges
into the stream the gate consumes. The topology is enumerable from published
manifests.

The component law (lexicon, petition entry): the approval topology is a DAG
anchored outside the component. Two agents approving each other satisfy the
per-agent law while jointly self-rooting; the law therefore binds the
connected component, and anchoring is checked at publish time by walking
declared grants — a validation, not runtime machinery.

The two approver kinds are dual, not interchangeable:

```text
human approver   witnessed — extra-systemic authority, no in-system
                 provenance; you cannot ask what the human read
agent approver   discharged — relayed authority, total provenance; the
                 approval, the approver, and the approver's inputs are all
                 immutable records, and the approver itself is diffable
```

## The Dial

The path is the second human-in-the-loop outlet, structurally distinct
from mid-turn tool gating: asynchronous (the denied turn keeps working),
between-turn, and over a better artifact — a declarative, diffable manifest
delta with provenance to the motivating denial. The human approves a grant,
not a spend; one decision amortizes over every future use.

Autonomy level is then a policy at one named point, not an architecture:

```text
full human      every petition requires the human anchor
policy-mixed    auto-approve inside a sub-anchor policy; escalate beyond it
full auto       agent approvers inside the tenant anchor; the human reads
                receipts after the fact
```

All three are the same wiring; only the gate's validation function changes.
Condensations ride the same dial; since a lawful condensation confers no new
capability, policy commonly auto-approves them while petitions still
escalate.

## Quiescence

Deny → petition → republish → retry is the strongest feedback loop in the
system, and chaotic couplings have no fixpoint. The petition controller
carries trigger quotas and depth limits (publishes per thread, per turn) as
kernel-enforced budget, per the trigger law. An agent that thrashes versions
exhausts a budget; it does not exhaust the operator.

## The First Mover

Autonomy in the strict sense is closure of the trigger graph: a system whose
activation chains need no witnessed event is its own first mover. That is
deliberately unconstructible here (lexicon trigger law): every activation
chain originates in a witnessed event.

What is constructible is the mandate — standing activation that is leased,
never owned: grantor, scope, budget, expiry, revocable. A mandate is
operationally indistinguishable from autonomy except at revocation time,
and revocation time is the point. Mission — manifest-resident, self-renewing
telos — is a banned word and a refused primitive. The self-check that
settled it: classifying the human as an agent inside the system would make
the component self-granting by reclassification alone, so the
witnessed/discharged distinction is the system boundary, and the boundary is
jurisdictional on purpose. You prompt the genie. That is the product.

## Boundary Summary

```text
widening    publish-gated, anchor-validated, receipted, next-bind effective
settling    publish-gated, capability-bounded by precedent, episode-pinned
narrowing   bind-layer, evidence-pinned, receipted, next-bind effective
approving   DAG of granted points, anchored outside the component
activating  witnessed origin always; standing purpose only as mandate
modeling    the anchor has no schema, ever
```

V1 ships none of this beyond the words and the laws; the paths build on
the petition controller, the anchoring validation, mandate records, the
recurrence projection, and the condensation and demotion controllers as
separate tickets once the V1 RC closes.
