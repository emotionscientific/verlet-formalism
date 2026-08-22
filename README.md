# Verlet Formalism

The working vocabulary and laws behind [Verlet](https://github.com/emotionscientific/verlet-kernel),
an experimental runtime for agents and workflows.

This is the canon the runtime is built against. It is published as it is
used: evolving, internally consistent, and checked on every commit. It is
not a specification in the standards sense and it is not finished. When the
code and this text disagree, one of them is wrong, and the fix lands here or
there with a commit that says which.

## What is here

| File | What it is |
| --- | --- |
| [`laws.md`](laws.md) | The laws that bind the system as a whole: the split between surfaces and couplings, the three laws, and the ladder of what is compiled into the kernel versus declared above it. Short. Read first. |
| [`lexicon.md`](lexicon.md) | Every primitive the runtime names, one entry each, with the word-level laws that travel with the word. New primitives are named here before they are implemented. |
| [`grounding/`](grounding/) | Longer background notes: the computer-science ancestry the design borrows from and departs from, the event and observation model, durable coordination and resume, and governed self-extension. Written July 2026; see the dating note at the top of each. |
| [`bin/canon-lint`](bin/canon-lint) | The mechanical check: canon files present, every relative link resolves, banned words absent from design prose. Runs in CI on every push. |

## How to read it

The laws fit on one screen. The lexicon is a reference, not a narrative;
open it when a word in the runtime's docs or code needs its exact meaning.
The grounding notes are for readers who want to know where an idea came
from and why the design does not simply reuse the ancestor.

If you have the runtime cloned, the shortest path is to ask a coding agent
in that checkout: "find where the lexicon entry for *binding* is implemented
and show me the attach and detach events." The text here is meant to be
checkable against the code, not trusted on its own.

## How this is built

Coding agents are used extensively in the development of Verlet, including
this document. Several different models draft, implement, and cross-examine
each other's claims, and a human architect makes the design decisions and
reviews what lands. This is what lets a very small team build a project of
this size at this speed, and it is why fixes land quickly after they are
found.

The formalism exists to hold the design intention fixed while the code
moves. It is the reference the agents are constrained by: the lexicon names
every primitive before it is implemented, the laws say what may not be
configured away, and the runtime is scanned against both on a regular
cadence. A scan that finds code and canon disagreeing produces either a fix
to the code or a struck line here, with a commit that says which.

Treat the text accordingly. It is current, it is checked, and it will
change.

## How it changes

Wrong content is struck directly by whoever finds it. Contested changes
(two defensible meanings with real consequences) are settled in discussion
and recorded here. There is no ratification ceremony. The git history is
the amendment log.

Contributions that fix a contradiction, a dead link, or a word used two
ways are welcome as pull requests. Proposals that add a primitive should
name it in the lexicon first and say what pain the name answers.

## License

CC BY 4.0. Cite as "Verlet Formalism, Emotion Scientific, <commit>".
