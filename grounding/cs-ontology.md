# The CS Ontology: Ancestry And Departures

> **Dating note.** Written July 2026, before the August 2026 authority recast.
> Where this document says *grant*, the current word is *binding* (an
> operation is attached to a thread by a recorded attach event and absent
> until then); who may attach is policy, not law. The laws and lexicon in
> this repository are current; this document is background and will be
> recast in place.

Status: internal grounding document — it coheres with
[the formalism](../formalism/README.md) and never overrides it. Vocabulary
defers to [the lexicon](../formalism/lexicon.md); the epistemic model lives in
[Streams, Couplings, And Context](event-streams-and-observations.md). This doc
names the computer-science ancestors of the Verlet design so contributors do
not re-derive them badly, and states precisely where Verlet departs from each.

The physics and philosophy ancestry (molecular dynamics, process philosophy,
semiotics) is documented alongside the epistemic model. This file is the CS
half of the family tree.

None of this is public positioning. Public docs say "agent platform." This doc
is for the people building the kernel.

## The Map

```text
the Ladder                 <- binding-time analysis / partial evaluation
agent = kernel(manifest)   <- first Futamura projection
compile/bind/publish       <- second Futamura projection
governed self-extension    <- the third projection, with a chaotic generator
condensation               <- the first projection of the content tower
demotion                   <- JIT deoptimization: guards, then deopt
product words              <- trait aliases / blanket impls over structural rows
coupling roles by sink     <- structural typing; ECS queries, not subclasses
the operation ABI          <- algebraic effects and handlers
the manifest               <- initial encoding; program as data
bind-time declarations     <- template metaprogramming; monomorphized surfaces
controllers, budgets       <- cybernetics; regulation under requisite variety
receipts as certificates   <- interactive proofs; Arthur and Merlin
the chaotic propagator     <- the departure: none of the ancestors had one
```

## Partial Evaluation And The Futamura Tower

Partial evaluation asks of every part of a program: when does this become
known? That question, applied to Verlet, *is* the Ladder:

```text
layer 0  bound at kernel compile time      (laws)
layer 1  bound at registration time        (contracts, versioned)
layer 2  bound at bind/turn time           (declarations, hot-swappable)
layer 3  bound at marketing time           (product words)
```

The compiled-versus-swappable decision rule ("if changing it could make a
receipt lie, it is compiled") is a binding-time assignment with an audit
constraint attached.

The tower, in Verlet terms:

```text
mix(kernel, manifest)  = agent          first projection
                                        the harness is the residual program:
                                        exportable, diffable, never declared

mix(mix, kernel)       = agent compiler second projection
                                        the compile/bind/publish path: the
                                        manifest is specialized once, with
                                        receipts, not re-interpreted per turn

mix(mix, mix)          = compiler generator
                                        third projection: hand it a new
                                        substrate, get the manifest compiler
                                        for that substrate
```

That tower is deterministic end to end: kernel and manifests, compile and
bind. A second tower runs on its residual. The running agent — model plus
resolved harness — is itself an interpreter, and what it interprets is task
knowledge in natural language: specs, skills, tickets, the recurring work of
a domain. The same projections apply one story up:

```text
mix(agent, recurring task)  = condensation   first projection: repeated
                                             interpretive work settles into
                                             a package (operation, fixtures,
                                             resources)
mix(mix, agent)             = the condenser  second projection: recurrence
                                             projection + condensation
                                             controller, a standing compiler
                                             from trajectories to packages
mix(mix, mix)               = the platform   third projection: every new
                                             agent gets its condenser from
                                             the substrate
```

The publish gate is the membrane between the stories: content-tower
residuals land in substrate terms only through it, and the three laws are
the membrane conditions. The upper tower is chaotic where the lower is
deterministic — which is why its proposals are facts on streams and its
acceptance is gated, never assumed.

The inference stack participates literally, not metaphorically. Prefix caching
is partial evaluation of the transformer on the static context: the KV cache
of the pinned sources is the residual program of specializing the model on the
manifest's identity block. Fine-tuning is specializing weights on history.
Inference systems have been doing Futamura projections by hand and calling
them optimizations.

Law this ancestry carries: specialization must be auditable. A residual
program with no receipt for how it was produced is folklore, not a harness.

The tower also has a gauge. Jones optimality — the partial-evaluation
benchmark that asks whether specialization removed the *entire* interpretive
layer — becomes measurable here, because every discharge carries its
coupling identity and every coupling is declaredly chaotic or deterministic.
The chaotic share of a thread's work, per recurring task family, is the
residual interpretive overhead: falling means condensation is removing
interpretation; a stable plateau is the genuinely judgmental remainder. The
metric is a view over receipts, and it is hard to game by construction: law
2 forbids chaos from claiming to be deterministic, and restart determinism
catches the lie on replay. Its per-family form is the recurrence signal that
triggers condensation — the collective variable the controller biases on.

And the departure that matters most, stated the same way as the cybernetics
flip below: partial evaluation covers the staging; the commitments that
remain after projecting onto the staging — truth, attestation, permission —
are the product. Nothing in the tower forces an anchor. That is why the wild
instances of self-specialization (prompt evolution, agent-design search,
self-improving scaffolds) are all self-rooted, and why governed
self-extension is not a Futamura corollary but a deontic addition to it.

## Traits, Typeclasses, And Structural Composition

The kernel defines few contracts: the operation ABI, the assembler contract,
the coupling quad. Product concepts are never implemented against directly.

"Memory" is a trait alias over a structural row:

```text
Memory = { writer: coupling, reader: pipeline source, index?: view }
```

Nothing ever `impl Memory`. Anything possessing the row *is* a memory system —
a blanket impl. The same holds for hooks (controller on lifecycle event
kinds), subagents (operations invoking operations over manifests and
threads), and skills (resource packages consumed by pipelines and tools).

Coupling roles are inferred, not declared. Propagator, projection, and
controller are determined by sink-relation — where the output plugs back in —
which makes the role a structurally assigned marker, not a nominal subtype.

The entity-component framing is the same move from a different industry: an
agent is an entity; couplings, sources, grants, and bindings are components;
product features are queries over components, not inheritance edges. "Supports
memory" is an archetype query.

Law this ancestry carries: one contract, many derived names. New product words
must lower to existing rows; a product word that needs a new kernel primitive
is a design failure until proven otherwise.

## Algebraic Effects And Handlers

The operation ABI is an effect system with capability checking.

An operation declares its effects — `net.http:POST:<origin>`,
`secret:<name>`, effect ports for durable writes — and cannot perform them. It
can only request them through the opaque invocation handle. The host is the
handler, chosen at invocation time. The receipt is the handler's evidence that
the effect was discharged.

This is why placement and provider swaps require no guest changes: same
program, different handler stack. It is also why authority composes safely
through recursion: a child invocation's grants are bounded by

```text
child grants  ⊆  parent grants  ∩  child's declared requirements
```

Nesting attenuates authority monotonically; no handler stack amplifies.

Departure from the ancestry: academic effect systems handle effects; Verlet
handles *and witnesses* them. An effect without a receipt did not lawfully
happen.

## Initial Encodings: The Program Is Data

The manifest is an AST. The kernel is its interpreter. This is the initial
encoding (free-monad / defunctionalized) school: represent the computation as
a value, keep interpreters swappable, enforce laws in the smart constructors.

The smart constructors are the compile/bind layer: alias resolution to
immutable hashes, fail-closed rejection of unknown refs, receipts for every
resolution. Hot-swap works because reconfiguration is handing the interpreter
a new term, never patching the interpreter.

Law this ancestry carries: the term language is total and inspectable. If a
behavior cannot be read off the declaration plus the receipts, it is not a
declaration — it is hidden interpreter state, and it is a bug.

## Template Metaprogramming At The Declaration Layer

The higher layers have the shape of template metaprogramming, and the analogy
is load-bearing rather than decorative:

- **Declarations are templates; bind is instantiation.** Layer-2 manifests are
  evaluated at bind time the way templates are evaluated at compile time:
  fully, before any run, producing a specialized artifact with no residual
  interpretation cost. Unknown refs fail at instantiation, not at runtime —
  the fail-closed compile is the hard instantiation error, deliberately
  without a SFINAE escape hatch.
- **Surfaces are monomorphization.** One operation contract instantiates per
  call boundary — CLI command, HTTP route, LLM tool, MCP export, bash builtin —
  each instance generated from the same definition and faithful to it. The
  surface law ("every surface is faithful, or it is illegal") is what keeps
  monomorphization from becoming specialization-with-divergence.
- **Layer-1 contracts are concepts.** The operation ABI shape, the assembler
  contract, and the selector grammar constrain what may instantiate, so
  instantiation errors are early, lawful, and stated in the contract's terms —
  not deep in the expansion.
- **Layer-3 words are template aliases.** `Memory = Coupling<writer> +
  Source<reader>` is a `using` declaration: a name for an instantiation
  pattern, adding no new machinery.
- **The harness is the expansion.** Exporting and diffing the harness is
  reading the instantiated template — the post-expansion artifact that shows
  what the declarations actually became.

Departure: C++ templates became an accidental Turing-complete language with no
laws, no budget, and famously unreadable expansion errors. Verlet caps the
declaration layer's power on purpose: declarations select and arrange among
registered instances; they do not compute. Anything that computes is a
coupling, where grants, provenance, and budgets apply. The expansion is
receipted, so reading it back is a feature, not an archaeology project.

## Cybernetics

The regulation vocabulary — controller, feedback through control streams,
budget-enforced quiescence — descends from cybernetics, and one theorem does
load-bearing work: Ashby's law of requisite variety. A regulator must hold at
least the variety of the system it regulates. A checklist rubric has fixed
variety and dies when the model's behavior space outgrows it; coherence is
relational, defined over the formalism rather than over outputs, so its
variety grows with the model consuming it. That is why the canon's acceptance test
is coherence (see [the loop](../formalism/loop.md)) and why quiescence
is enforced by budget, never assumed from convergence.

The containment relation flips by axis, and the flip is the content.
Extensionally, cybernetics is the superset: every Verlet deployment is a
cybernetic system; almost no cybernetic system is a lawful Verlet one.
Intensionally it is closer to a subset: cybernetics has no concept of truth
(an authoritative, append-only record of what happened), no attestation
(witnessed versus discharged), no permission (a grant is deontic — what a
part *may* do; cybernetic constraint is only descriptive — what it *can*
do), and no binding time. Cybernetics covers the dynamics; the commitments
that remain after projecting onto the dynamics are the product.

Departure: even second-order cybernetics, which drew the observer inside the
system, left observation as signal. Verlet makes the act of observation a
durable event with provenance. The regulator must leave receipts — Arthur,
not merely a thermostat.

## Interactive Proofs: Receipts As Certificates

The trust architecture is the prover/verifier asymmetry of complexity
theory: solutions hard to generate, cheap to check. The model is Merlin — an
untrusted, unboundedly clever prover (Babai's Arthur–Merlin games, 1985);
the kernel is Arthur — a weak, honest, deterministic verifier. A receipt is
a certificate: the witness that lets the deterministic side accept
nondeterministic work without reproducing it. The publish gate imports
untrusted computation into the trusted record by demanding checkable
witnesses — proof-carrying code makes the same move for mobile code.

The hope theorem is IP = PSPACE (Shamir, 1992): a polynomially bounded
verifier, given interaction and randomness, can verify far beyond what it
could ever compute. Weak honest Arthur can govern arbitrarily strong
untrusted Merlin. That is the theorem-shaped form of the whole bet.

The event/view distinction is this boundary drawn inside the system. A view
claims membership in P: deterministic, regenerable, deletable. An event is
the certificate record of computation that cannot be recomputed, only
witnessed. Law 2 — chaotic output discharges as events, never views — reads:
chaos may never claim to be in P. The view law's converse ("if deleting it
would lose truth, it was an event that got misfiled") reads: what cannot be
recomputed needs a certificate. The formalism loop's ratchet is then the
migration of predicates from Merlin to Arthur — from coherence-judged to
certificate-checked.

Departures, so the analogy never overclaims: an NTM's nondeterminism is
angelic — it finds the accepting path by definition; a model merely samples,
so Merlin sometimes proves nothing, and budgets exist because there is no
fixpoint to converge to. And the construction stands on the *empirical*
generation/verification gap, not on the P ≠ NP conjecture; it monetizes the
asymmetry whether or not the Clay prize is ever claimed.

## The Departure: A Chaotic Generator In The Tower

Every ancestor above assumes its functions are deterministic. Typeclass
resolution, effect handling, template instantiation, partial evaluation — all
of them are meaningful because re-running them reproduces the result.

Verlet has a typed slot for a function that violates this: the chaotic
coupling, the model-backed function whose output is not reproducible from its
inputs. None of the ancestors needed one, because none of them had an LLM in
the loop. The three laws are exactly the conditions under which a chaotic
generator can sit inside a specialization tower without the tower becoming
folklore:

```text
1. chaotic output discharges as events, never views
   (proposals are facts on streams, not mutations)
2. every discharge carries provenance
   (each rung of the tower has a chain back to whoever asked)
3. specialization itself stays deterministic, budgeted, receipted
   (the chaos is quarantined in the proposal step; mix stays lawful)
```

with quiescence enforced by budget, because chaotic couplings have no
fixpoint. Trigger quotas and depth limits are what keep self-application from
being a fork bomb: the third projection with a thermostat.

This places Verlet between the two failure modes in the wild:

```text
Gödel machine        prove improvement before every self-modification
                     formal, unbuildable

GEPA / fine-tuning   self-modify with no durable ground truth, no receipts,
                     no reproducible evaluation
                     practical, unaccountable; cannot hill-climb a landscape
                     it cannot stand still on

Verlet              require proof of provenance, not proof of improvement
                     receipts, not theorems: every step recorded, attributed,
                     reproducible in its deterministic parts, and reversible
```

Two corollaries close the loop on the middle failure mode. Receipts make the
landscape rigid enough to hill-climb: with provenance and replay,
compile-time search over candidate condensations becomes admissible, and the
gate is the selection operator. And the tower keeps a way down: demotion is
the JIT deoptimization of this construction — a residual whose guards fail
returns to interpretive execution, receipted, instead of rotting in place.

Restart determinism, the event-kind freeze, origin and provenance fields, and
receipts-as-events are therefore not release hygiene. They are the load-bearing
conditions of the whole construction, which is why they gate V1.

## What This Doc Is Not

- Not canon: this doc grounds [the formalism](../formalism/README.md); it never
  overrides it.
- Not vocabulary: words and their laws live in the lexicon.
- Not the epistemic model: streams, couplings, and context live in their doc.
- Not positioning: no public material should cite Futamura, effects, or
  metadynamics; the public dictionary is the lexicon's Public Jargon table.
- Not a claim of equivalence: each ancestry holds where stated and breaks at
  the chaotic coupling, and the breaks are listed next to the inheritances.
