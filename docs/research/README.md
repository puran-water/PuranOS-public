# Research That Shaped the Architecture

## Why this exists

PuranOS is not built from first principles or from industry convention.
It is built from specific research findings about agent coordination,
memory systems, and enterprise workflow architecture — then validated
against the requirements of industrial process engineering.

This section documents what we learned and how it shaped design decisions.
The goal is transparency: the reader should understand not just what
PuranOS does, but why specific architectural choices were made and what
evidence supports them.

## The central question

How should an AI-native industrial firm organize its data, coordinate its
agents, and capture its expertise?

The conventional answer is some combination of: RAG over documents,
vector-store memory, multi-agent swarms, and prompt engineering. The
research suggests a different answer — one based on explicit state, typed
schemas, structured coordination, and procedural knowledge capture.

## Key research threads

### Agent coordination and state management

The llmenron exchange (strangeloopcanon/llmenron) ran four waves of
experiments stress-testing agent architectures. The findings were
unambiguous: explicit state beats scratchpad memory, shared boards
improve quality for both single and multi-agent setups, and actor
identity eliminates unauthorized actions.

These findings directly shaped PuranOS's use of OpenProject as the
coordination substrate, the hybrid state model (OpenProject for
long-running state, PostgreSQL for execution reliability), and the
persona boundary enforcement model.

Detailed treatment: [Coordination and State](coordination-and-state.md)

### Schema'd state versus memory systems

Research on enterprise AI memory systems consistently shows that
structured, schema'd state outperforms unstructured memory.
StructMemEval found that memory helps only when organized into
task-appropriate structures. "Lost in the Middle" showed that retrieval
degrades in long contexts. StateFlow showed that explicit state machines
beat ReAct-style reasoning at lower cost.

These findings directly shaped PuranOS's "schema over memory" thesis:
the enterprise's own tools and purpose-built schemas provide a better
knowledge substrate than vector stores, key-value caches, or RAG
systems.

Detailed treatment: [Schema Over Memory](schema-over-memory.md)

### Counter-evidence PuranOS takes seriously

The research is not uniformly positive about multi-agent systems or AI
coordination:

- Multi-agent systems show high failure rates in studied deployments.
- Single agents can match multi-agent performance at lower cost in many
  scenarios.
- 79% of production agent systems use manual prompt construction, not
  frameworks.
- The best model/config combination achieves only 70.8% on simpler
  enterprise tasks.

PuranOS does not ignore this evidence. It responds with specific design
choices: function-calling over ReAct, shallow workflows with human
gates, multiple agents only where domain heterogeneity requires them,
and a shared board as the coordination primitive.

Detailed treatment: both research documents above include
counter-evidence sections.

## How to read these documents

Each research document follows a consistent structure:

1. The research finding and its source.
2. What it means for enterprise AI architecture.
3. How PuranOS applies the finding.
4. Counter-evidence and how PuranOS responds.

The documents cite findings and conclusions, not just paper names. The
reader should come away understanding the evidence, not just knowing
that evidence exists.

## Research documents

- [Coordination and State](coordination-and-state.md) — What the
  research says about agent coordination, shared state, and multi-agent
  systems.
- [Schema Over Memory](schema-over-memory.md) — Why schema'd state
  outperforms memory systems and RAG for enterprise AI.

## Related approach documents

These approach documents apply the research findings to PuranOS's
architecture:

- [Schema Over Memory (Approach)](../approach/schema-over-memory.md) —
  The flagship thesis document.
- [Coordination Substrate](../approach/coordination-substrate.md) —
  OpenProject as shared board.
- [Skills as Expertise](../approach/skills-as-expertise.md) —
  Procedural knowledge capture.
