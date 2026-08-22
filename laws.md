# Laws

Status: canonical, part of [the formalism](README.md). These are the laws
that bind the system as a whole; word-level laws live with their words in
[the lexicon](lexicon.md). Wrong content here gets struck directly
(README, How It Changes); a contested change is a plain question for the
anchor.

## The Split

There are exactly two kinds of mechanism in Verlet:

```text
surfaces re-present.   A surface shows the same contract somewhere else.
couplings transform.   A coupling makes something new happen.
```

## The Three Laws

```text
1. Every surface is faithful, or it is illegal.
2. Every discharge carries provenance; chaotic output discharges as events,
   never as views.
3. Assembly is deterministic, budgeted, and receipted; it selects and
   arranges, never creates.
```

## The Ladder: Compiled Versus Swappable

```text
layer 0  laws            compiled into the kernel; not configurable
         stream order, event immutability, origin and provenance,
         attachment and mediation (a tool not attached to a thread does
         not exist for it; every external effect passes a
         runtime-mediated surface), receipts, budget-enforced
         quiescence, the three laws

layer 1  contracts       shape compiled; instances registered and versioned
         operation ABI, coupling shape, assembler contract, selector
         grammar, event-kind vocabulary, surface boundaries

layer 2  declarations    config; versioned; hot-swappable between turns
         manifests, pipelines, model profiles, tool declarations,
         policies, bindings, aliases

layer 3  product language must lower to layers 0-2
         memory, skills, hooks, subagents, personas, harness
```

The decision rule for what stays compiled:

```text
If changing it could make a receipt lie, it is compiled.
If changing it only changes future behavior, and the change is itself
receipted, it is swappable.
```

Robustness comes from layer 0 being small and immovable. Flexibility comes
from everything above it being named, versioned, and replaceable. The kernel
compiles the laws; declarations pick the instances; couplings do the work.

## Carried Laws

Laws the ancestry carries, stated canonically here; the narratives that
motivate them live in [the CS ontology](grounding/cs-ontology.md):

- Specialization is auditable: a residual program with no receipt for how
  it was produced is folklore, not a harness.
- An effect without a receipt did not lawfully happen.
- One contract, many derived names: a product word that needs a new kernel
  primitive is a design failure until proven otherwise.
- The term language is total and inspectable: behavior that cannot be read
  off the declaration plus the receipts is hidden interpreter state, and a
  bug.
- Declarations select and arrange among registered instances; they never
  compute. Anything that computes is a coupling, where provenance and
  budgets apply.
