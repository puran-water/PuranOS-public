# Schema First: Why Structured State Outperforms Memory Systems

> Research evidence for PuranOS's central architectural thesis:
> a properly schema'd database substrate outperforms memory/RAG/vector stores
> for enterprise AI in domains with known ontologies. Memory remains appropriate
> for unstructured residue and conversational continuity.

---

## Why this exists

PuranOS is built on the premise that enterprise applications — CMMS, ERP,
project management, procurement — already constitute a structured knowledge
substrate. Most AI platforms ignore this and bolt on a separate "memory layer"
(vector stores, key-value caches, free-form logs). This document assembles the
research evidence that this bolt-on approach is architecturally inferior to
querying schema'd application state directly.

The companion document [Schema Over Memory (Approach)](../approach/schema-over-memory.md)
presents the thesis in architectural terms. This document presents the empirical
basis.

---

## The memory problem at enterprise scale

Memory-based AI architectures attempt to solve enterprise knowledge with
retrieval layers. They extract facts from conversations, store embeddings in
vector databases, and retrieve chunks at inference time. This works at prototype
scale: a few hundred documents, a handful of users, a limited domain.

At enterprise scale, these approaches fail in predictable ways.


### Context bloat

**[Lost in the Middle](https://arxiv.org/abs/2307.03172) (Liu et al., 2023 — peer-reviewed)**
demonstrated that LLM performance on retrieval tasks degrades when relevant
information sits in the middle of long contexts. Models reliably find information
at the beginning or end of their context window but struggle with information
positioned centrally.

This directly undermines RAG approaches that retrieve and concatenate chunks.
More retrieved context is not reliably better. It can actively degrade
performance by burying relevant information among irrelevant chunks.

Schema'd queries (SQL, typed API calls) avoid this entirely. They return exactly
the relevant data in a predictable structure. There is no "middle" to get lost
in.

| Approach | Retrieval mechanism | Vulnerable to positional bias |
|---|---|---|
| RAG (chunk concatenation) | Embedding similarity, top-k | Yes — "Lost in the Middle" effect |
| Long-context stuffing | Full document injection | Yes — performance degrades with length |
| Schema'd query | SQL / typed API call | No — returns exact result set |


### Garbage accumulation

Free-form memory systems accumulate observations without principled curation.
Every conversation, every tool call, every intermediate result is a candidate
for storage. Without strong extraction, deduplication, and pruning, the memory
fills with low-quality, redundant, or contradictory entries.

This is difficult to solve at scale through engineering alone. It is an inherent property of
unstructured storage. The more you store, the harder retrieval becomes, and the
more likely you are to retrieve garbage alongside signal.

Schema'd systems avoid this by construction. A work order in the CMMS either has
the required fields or it does not. There is no "fuzzy" work order. There is no
duplicate unless the application has a bug. The schema IS the curation mechanism.


### Ontology drift

Free-form extraction cannot maintain consistency across thousands of facts. Is
it "RO membrane" or "reverse osmosis membrane" or "RO element"? In a vector
store, all three are separate entries with slightly different embeddings. In a
schema'd system, there is one equipment type code that maps to all three
surface forms.

Over time, unstructured memory develops internal inconsistencies that are
invisible to the agents using it. Two agents may retrieve different
representations of the same concept and produce contradictory outputs without
any mechanism to detect the conflict.

| Failure mode | Unstructured memory | Schema'd system |
|---|---|---|
| Synonyms stored as distinct entities | Common, undetectable | Prevented by controlled vocabulary |
| Contradictory facts coexist | Common, no resolution mechanism | Prevented by unique constraints |
| Stale facts persist alongside updates | Common, no expiry mechanism | Prevented by update-in-place semantics |


### No write path

RAG retrieves but cannot update the source of truth. When an agent learns that
a vendor has changed their pricing, it cannot update the procurement database
through the RAG system. It can only store a new memory that "vendor X now
charges Y." The original source remains unchanged, creating a split-brain
problem.

Schema'd systems have native write paths. An agent with procurement tool access
can update a vendor quote directly. The update is validated, timestamped, and
immediately visible to every other agent that queries the schema.

This asymmetry — read-only retrieval versus read-write participation — is
fundamental. Agents that can only read memory are observers. Agents that can
write to schema'd applications are participants.


### Duplication of existing application state

The most fundamental problem: a separate memory layer duplicates what enterprise
applications already store.

When you deploy OpenProject, the work packages ARE the task memory. When you
deploy a CMMS, the work orders ARE the maintenance memory. When you deploy a
procurement system, the purchase orders ARE the commercial memory.

Building a parallel memory system that extracts and stores the same information
is architecturally redundant. It creates a synchronization problem where none
existed.

---

## Research evidence

### StructMemEval (February 2026 preprint) — [preprint](https://arxiv.org/abs/2602.11243)

Evaluated structured versus unstructured memory for AI agents across multiple
task types. The benchmark tests agents on tasks requiring persistent state
management over extended interactions. (Note: this is a February 2026 preprint /
work in progress, not yet peer-reviewed.)

Key finding: structured memory outperforms unstructured memory only when
organized into task-appropriate structure. Ledgers for financial tracking. Trees
for hierarchical data. Trackers for state management.

Free-form memory blobs do not reliably outperform simple context windows. The
structure must match the domain. Importantly, the paper does not prove that
application schemas universally dominate memory — it shows that memory systems
work when structure matches task structure.

| Memory type | When it helps | When it fails |
|---|---|---|
| Unstructured blob | Short sessions, simple recall | Multi-session, cross-referencing |
| Generic structured | Moderate improvement over blob | Domain mismatch degrades performance |
| Task-appropriate structure | Significant improvement | Requires domain expertise to design |

**Implication for PuranOS:** Application schemas ARE task-appropriate structure.
A procurement schema is a purpose-built ledger for cost tracking. An OpenProject
hierarchy IS a tree for work decomposition. A plant-state model IS a tracker for
engineering state. These are not generic "structured memory." They are
domain-specific structure built by experts in each domain.


### Agent Workflow Memory (Wang et al., 2024) — [preprint](https://arxiv.org/abs/2409.07429)

Compared episodic memory (replay of past interactions) against procedural memory
(reusable workflows extracted from successful task completions).

Results across two major benchmarks:

| Benchmark | Episodic memory | Procedural memory | Relative improvement |
|---|---|---|---|
| Mind2Web | baseline | +24.6% | Procedural wins |
| WebArena | baseline | +51.1% | Procedural wins |

Procedural memory — encoding HOW to do something rather than WHAT happened last
time — transfers more reliably across tasks.

**Implication for PuranOS:** Skills are procedural memory. A skill for sizing an
MLE bioreactor works across all MLE projects. Episodic memory of one specific
MLE sizing may or may not transfer. See [Ontology Layers](../architecture/ontology-layers/README.md)
for how skills, schemas, and session context compose.


### StateFlow (Wu et al., 2024) — [preprint](https://arxiv.org/abs/2403.11322)

Compared explicit state machine-based agent workflows against ReAct-style
free-form reasoning.

| Metric | ReAct (baseline) | StateFlow |
|---|---|---|
| Success rate | 1.0x | +13-28% relative |
| Token cost | 1.0x | 0.20-0.33x |

Explicit state transitions outperform free-form reasoning at dramatically lower
cost. The agent does not need to reason about what state it is in when the state
machine tracks that externally.

**Implication for PuranOS:** OpenProject status workflows are explicit state
machines. Work packages move through defined states (New, In Progress, In
Review, Closed). The execution ledger records state transitions. Neither relies
on agents remembering what state they are in. The application enforces it.


### AgentSM: Semantic memory for enterprise reasoning (Jan 2026) — [preprint](https://arxiv.org/abs/2601.15709)

AgentSM introduces semantic memory for text-to-SQL tasks, storing structured
programs rather than raw text. The system retrieves previously successful query
structures and adapts them to new questions, bridging the gap between
unstructured user intent and structured database operations.

**Implication for PuranOS:** AgentSM validates the principle that schema-heavy
enterprise reasoning benefits from structured program storage rather than raw
memory. PuranOS skills serve a similar function — they encode structured
procedures (tool call sequences, parameter mappings) that map domain intent to
schema'd operations. The difference is that PuranOS encodes these as
expert-curated skills rather than auto-extracted query patterns.


### Context compression literature

Multiple studies (Ge et al. 2023; Jiang et al. 2023; Chevalier et al. 2023)
show that pruning and compressing conversation history cuts token costs while
preserving or improving task accuracy.

The implication is revealing: much of what agents carry in their context window
is not load-bearing. It is noise accumulated over the course of a conversation.

Schema'd queries avoid the compression problem entirely. An agent that queries
the procurement schema for vendor quotes on RO membranes gets exactly the
relevant data. Not a compressed version of everything it has ever discussed
about procurement. Not a lossy summary. The exact rows.

**[Beyond a Million Tokens](https://arxiv.org/abs/2510.27246) (ICLR 2026 —
peer-reviewed)** introduced the BEAM benchmark testing up to 10M tokens. At
these scales, context compression and selective retrieval become essential —
no model can attend uniformly over 10M tokens. This reinforces the schema-first
approach: for structured enterprise data, querying the schema returns exactly
the needed rows regardless of how much total data exists. The 10M-token problem
simply does not arise when the retrieval mechanism is a typed query, not a
context window.


### Long-context versus memory (2024-2026)

Comparative studies on LoCoMo and LongMemEval benchmarks found that long context
is more accurate than fact-extraction memory but more expensive. Memory-based
approaches are cheaper but less accurate.

**[AMA-Bench](https://arxiv.org/abs/2602.22769) (preprint, Feb 2026)** provides
a long-horizon memory benchmark and finds that many purpose-built memory systems
underperform long-context baselines. This is a striking result: the specialized
memory systems designed to solve the long-context problem often do not beat
simply stuffing the full history into the context window.

**[Beyond the Context Window](https://arxiv.org/abs/2603.04814) (preprint, Mar
2026)** directly compared Mem0 (a fact-extraction memory system) against
long-context GPT-5-mini. Long-context GPT-5-mini outperformed Mem0 on
LongMemEval and LoCoMo benchmarks. Memory becomes cost-advantaged only after
enough turns accumulate.

| Approach | Accuracy | Cost at 10 turns | Cost at 100+ turns |
|---|---|---|---|
| Long context (full history) | Highest | Moderate | Very high |
| Fact-extraction memory | Lower (often worse than long-context baselines per AMA-Bench) | Lower | Lower |
| Schema'd query (PuranOS extrapolation — not experimentally measured) | Expected highest for structured data | Expected lowest | Expected lowest |

The third row is a PuranOS architectural inference, not a published experimental
result. The research directly supports skepticism toward both naive long-context
stuffing and purpose-built memory systems. The evidence supports **schema first
for structured enterprise state; memory for unstructured residue and
conversational continuity** — not a blanket anti-memory conclusion.

**Implication for PuranOS:** For data already structured in a database, schema'd
queries should be both more accurate and cheaper than either approach. For
unstructured conversational context and residual knowledge, memory approaches
remain appropriate.

---

## Enterprise software IS memory

This is the core insight that the research supports but does not state directly.

When you write a work order in Atlas CMMS, that IS the memory of what
maintenance was done. Who did it, when, what parts were used, what was found.

When you create a purchase order in InvenTree, that IS the memory of what was
ordered. From which vendor, at what price, for which project.

When you update a work package in OpenProject, that IS the memory of project
progress. What was done, what remains, who is responsible.

When you record a cost observation in the procurement schema, that IS the memory
of equipment pricing. Timestamped, CEPCI-indexed, vendor-attributed.

Each of these applications already provides:

| Capability | Enterprise application | Memory layer (reimplemented) |
|---|---|---|
| Validation | Required fields, type constraints, FK relationships | Best-effort extraction heuristics |
| Permissions | Role-based access control | Typically absent or bolt-on |
| Audit history | Who changed what, when | Append-only log, no correction mechanism |
| Domain ontology | Schema defines entities and relationships | Emergent, inconsistent, drifting |
| Write path | Native CRUD operations | Read-only retrieval |

Building a separate memory layer that extracts facts from these applications,
stores them in a vector database, and retrieves them for agents is redundant
work. It takes structured data, destructures it into embeddings, and then
attempts to re-structure it at retrieval time.

---

## The three-layer model

PuranOS uses three persistence layers, each serving a distinct temporal and
structural purpose.

| Layer | What it stores | Implementation |
|---|---|---|
| Session persistence | Conversational continuity within a task | Per-thread context (email thread, project session) |
| PM tool persistence | Task-level state: owner, status, blockers, approvals, due dates | OpenProject work packages |
| Application schemas | Domain-level facts: equipment specs, vendor quotes, maintenance history, plant parameters | Postgres-backed tools (enterprise OSS + purpose-built) |

Together, these three layers provide conversational continuity, task tracking,
and domain knowledge — without garbage accumulation, ontology drift, or
maintenance burden.

### Session persistence

Handles the "what were we just talking about" problem. Scoped to a single task
thread. Discarded or archived when the task completes. This is the only layer
where unstructured text is appropriate, because conversational context is
inherently unstructured and short-lived.

### PM tool persistence

Handles the "what is the state of this project" problem. OpenProject work
packages carry status, assignments, dependencies, due dates, and custom fields.
Agents query this layer to understand what needs doing. They write to it to
record progress. The PM tool enforces workflow constraints (e.g., a work package
cannot move to "Closed" without review).

### Application schemas

Handles the "what do we know about this domain" problem. Equipment databases,
vendor catalogs, maintenance histories, regulatory requirements, design
parameters. Each application has a purpose-built schema maintained by domain
experts. Agents access these through MCP tool calls that map to validated API
endpoints.

See [Coordination and State](coordination-and-state.md) for how agents
coordinate across these layers.

---

## Counter-arguments and scope limits

This document does not argue that:

**Memory systems are never useful.** For truly unstructured domains with no
known ontology, memory approaches may be the best available option. Not every
domain has the decades of standardization that industrial process engineering
has. Even in structured domains, conversational continuity and unstructured
residue (e.g., meeting notes, informal context) benefit from memory approaches.

**RAG has no place.** Document retrieval over unstructured text — regulations,
specifications, vendor literature — remains valuable when the source material is
inherently unstructured. PuranOS uses document retrieval for regulatory text
(eCFR, state permits) where the source does not have a structured
representation. The argument is against using RAG as the primary knowledge
substrate when structured alternatives exist.

**Context windows do not matter.** Session persistence matters. Context
management within a session is a real engineering challenge. The argument is
that this challenge should not be conflated with the knowledge management
challenge.

**Schema design is easy.** Purpose-built schemas require domain expertise and
evolve over time. The investment in schema design is substantial. The argument
is that this investment pays off relative to the alternative of maintaining a
memory extraction pipeline.

---

## Summary of evidence

| Claim | Supporting evidence | Evidence tier | Mechanism |
|---|---|---|---|
| RAG degrades with scale | [Lost in the Middle](https://arxiv.org/abs/2307.03172) (Liu 2023) | Peer-reviewed | Positional bias in attention |
| Structured > unstructured memory (when structure matches domain) | [StructMemEval](https://arxiv.org/abs/2602.11243) (Feb 2026) | Preprint | Task-appropriate structure matches domain |
| Procedural > episodic memory | [Agent Workflow Memory](https://arxiv.org/abs/2409.07429) (Wang 2024) | Preprint | Workflows transfer; episodes may not |
| State machines > free-form reasoning | [StateFlow](https://arxiv.org/abs/2403.11322) (Wu 2024) | Preprint | Explicit transitions reduce error and cost |
| Context compression helps; extreme scale needs selective retrieval | Multiple (2023-2024); [Beyond a Million Tokens](https://arxiv.org/abs/2510.27246) (ICLR 2026) | Mixed (peer-reviewed + preprint) | Most context is not load-bearing; 10M tokens exceeds attention capacity |
| Long context > memory but costlier; many memory systems underperform baselines | [AMA-Bench](https://arxiv.org/abs/2602.22769) (Feb 2026); [Beyond the Context Window](https://arxiv.org/abs/2603.04814) (Mar 2026) | Preprint | Schema'd queries expected to avoid the trade-off |
| Schema-heavy reasoning benefits from structured program storage | [AgentSM](https://arxiv.org/abs/2601.15709) (Jan 2026) | Preprint | Structured programs > raw text for enterprise reasoning |
| Enterprise workflows need domain-specific evaluation | [WoW-bench](https://arxiv.org/abs/2601.22130); [Agent-Diff](https://arxiv.org/abs/2602.11224); [FireBench](https://arxiv.org/abs/2603.04857) | Preprint | Toy benchmarks miss enterprise complexity |

The convergent finding: for domains with known ontologies, schema-first
approaches outperform retrieval-based approaches. When the structure already
exists in enterprise applications, building a parallel unstructured retrieval
layer is redundant. Memory remains appropriate for unstructured conversational
context, residual knowledge, and domains without established schemas.

Note: this evidence stack includes peer-reviewed publications (Lost in the
Middle, Beyond a Million Tokens at ICLR 2026, Why Do Multi-Agent LLM Systems
Fail? at NeurIPS 2025), a peer-reviewed journal paper
([QSDsan](https://pubs.rsc.org/en/content/articlelanding/2022/ew/d2ew00455k),
2022), a technical report ([MAP](https://arxiv.org/abs/2512.04123), 2025), and
multiple arXiv preprints. The conclusions are research-informed, not
research-settled.

---

## Further reading

- [Schema Over Memory (Approach)](../approach/schema-over-memory.md) — The flagship thesis document
- [Ontology Layers](../architecture/ontology-layers/README.md) — The three ontology sources in detail
- [Coordination and State](coordination-and-state.md) — Related research on agent coordination
