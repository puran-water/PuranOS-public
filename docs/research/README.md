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

**A note on evidence quality:** The evidence cited here spans multiple
tiers: peer-reviewed publications (e.g., [Lost in the Middle](https://arxiv.org/abs/2307.03172),
[QSDsan](https://pubs.rsc.org/en/content/articlelanding/2022/ew/d2ew00455k),
[Beyond a Million Tokens](https://arxiv.org/abs/2510.27246) at ICLR 2026,
[Why Do Multi-Agent LLM Systems Fail?](https://openreview.net/forum?id=fAjbYBmonr)
at NeurIPS 2025), technical reports
([MAP](https://arxiv.org/abs/2512.04123)), arXiv preprints, and experiment
repositories ([LLMenron](https://github.com/strangeloopcanon/llmenron)).
Each citation is tagged with its evidence tier. The conclusions are
research-informed, not research-settled. Where findings come from
peer-reviewed publications, that is noted. Where they come from preprints
or experiment repos, the inferential weight should be calibrated
accordingly.

## The central question

How should an AI-native industrial firm organize its data, coordinate its
agents, and capture its expertise?

The conventional answer is some combination of: RAG over documents,
vector-store memory, multi-agent swarms, and prompt engineering. The
research suggests a different answer — one based on explicit state, typed
schemas, structured coordination, and procedural knowledge capture.

## Key research threads

### Agent coordination and state management

The [LLMenron exchange](https://github.com/strangeloopcanon/llmenron)
(repo experiment, not peer-reviewed) ran four waves of experiments
stress-testing agent architectures. The findings were unambiguous:
explicit state beats scratchpad memory, shared boards improve quality for
both single and multi-agent setups, and actor identity eliminates
unauthorized actions.

These findings directly shaped PuranOS's use of OpenProject as the
coordination substrate, the hybrid state model (OpenProject for
long-running state, PostgreSQL for execution reliability), and the
persona boundary enforcement model.

A growing body of enterprise-specific benchmarks now tests agent
performance on real business workflows:
[WoW-bench](https://arxiv.org/abs/2601.22130) (preprint, Jan 2026)
evaluates against 4,000+ ServiceNow business rules across 55 workflows;
[Agent-Diff](https://arxiv.org/abs/2602.11224) (preprint, Feb 2026)
benchmarks 224 enterprise API tasks with state-diff evaluation;
[FireBench](https://arxiv.org/abs/2603.04857) (preprint, Mar 2026)
tests enterprise instruction-following across 2,400+ samples. These
benchmarks confirm that enterprise agent evaluation is maturing beyond
toy tasks, though all three are preprints awaiting peer review.

Detailed treatment: [Coordination and State](coordination-and-state.md)

### Schema'd state versus memory systems

Research on enterprise AI memory systems consistently shows that
structured, schema'd state outperforms unstructured memory for domains
with known ontologies — while memory remains appropriate for unstructured
residue and conversational continuity.
[StructMemEval](https://arxiv.org/abs/2602.11243) (February 2026
preprint) found that memory helps only when organized into
task-appropriate structures. [Lost in the Middle](https://arxiv.org/abs/2307.03172)
(peer-reviewed) showed that retrieval degrades in long contexts.
[StateFlow](https://arxiv.org/abs/2403.11322) (preprint) showed that
explicit state machines beat ReAct-style reasoning at lower cost.

These findings directly shaped PuranOS's "schema over memory" thesis:
the enterprise's own tools and purpose-built schemas provide a better
knowledge substrate than vector stores, key-value caches, or RAG
systems — schema first for structured enterprise state; memory for
unstructured residue and conversational continuity.

Detailed treatment: [Schema Over Memory](schema-over-memory.md)

### Skills as expertise: mixed evidence

[Voyager](https://arxiv.org/abs/2305.16291) (preprint) and [Agent
Workflow Memory](https://arxiv.org/abs/2409.07429) (preprint) showed
that procedural memory — reusable skills and workflows — outperforms
episodic replay for cross-task generalization. However, newer benchmarks
add important nuance: [SkillsBench](https://arxiv.org/abs/2602.12670)
(preprint, Feb 2026) found that curated skills improve performance by
+16.2 percentage points across 84 tasks, but self-generated skills
provide no benefit. [SWE-Skills-Bench](https://arxiv.org/abs/2603.15401)
(preprint, work-in-progress, Mar 2026) tested 49 SWE skills and found an
average improvement of only +1.2pp, with 39 of 49 skills showing zero
improvement. The implication: skill curation and narrow targeting matter
more than skill quantity. PuranOS's approach of domain-expert-curated
skills aligns with this evidence.

### Counter-evidence PuranOS takes seriously

The research is not uniformly positive about multi-agent systems or AI
coordination:

- Multi-agent systems show high failure rates in studied deployments.
  [Why Do Multi-Agent LLM Systems Fail?](https://openreview.net/forum?id=fAjbYBmonr)
  (peer-reviewed, NeurIPS 2025 Datasets & Benchmarks spotlight)
  catalogued 14 failure modes across 1,600+ traces.
- Single agents can match multi-agent performance at lower cost in many
  scenarios.
- [MAP](https://arxiv.org/abs/2512.04123) (2025 technical report)
  surveyed 306 practitioners and 86 deployed systems: 79% use manual
  prompt construction, not frameworks; 68% limit agents to 10 or fewer
  steps.
- [AgentArch](https://arxiv.org/abs/2509.10769) (2025 preprint) tested
  18 architectures on enterprise tasks: the best model/config combination
  achieves only 70.8% on simpler tasks and 35.3% on harder ones.

PuranOS does not ignore this evidence. It responds with specific design
choices: function-calling over ReAct, shallow workflows with human
gates, multiple agents only where domain heterogeneity requires them,
and a shared board as the coordination primitive.

Detailed treatment: both research documents above include
counter-evidence sections.

## How to read these documents

Each research document follows a consistent structure:

1. The research finding and its source (with evidence tier tag).
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
