# The Primacy of Schema'd State

## Why this exists

Most AI systems try to solve the "enterprise knowledge" problem by building
memory layers — vector stores, key-value caches, free-form logs — or RAG
systems over existing documents. At prototype scale, these work. At enterprise
scale, they produce context bloat and "garbage memory."

The alternative: make the enterprise's own tools and schemas the primary
knowledge substrate. When an engineer writes a work order in the CMMS, that IS
the memory of what maintenance was done. When a project manager creates an
opportunity in the CRM, that IS the memory of the commercial relationship. A
separate "memory layer" is architecturally redundant.

Schema'd state is the authority for canonical objects — equipment, streams,
tasks, costs. When unstructured documents matter — vendor submittals, permits,
field reports — they are served via document management with IDs linked from
schema'd records, so retrieval always returns evidence tied to the object it
concerns, not floating chunks of text. The pattern is not "schema instead of
retrieval" but "schema for canonical state, retrieval for evidentiary context,
with explicit joins between them."

PuranOS assembles its ontology from three source types. Each brings schema —
validated, typed, Postgres-backed — not free-form text.


---


## The three ontology sources

### Source 1: Self-hosted enterprise OSS

When you deploy OpenProject, you get a Postgres schema for work packages,
projects, users, types, statuses, priorities, custom fields, relations, and
attachments. When you deploy Twenty CRM, you get a schema for contacts,
companies, opportunities, activities, and pipelines. When you deploy InvenTree,
you get a schema for parts, stock, BOMs, suppliers, and purchase orders. When
you deploy Atlas CMMS, you get a schema for assets, locations, work orders,
preventive maintenance, and meters.

Each of these applications enforces its domain model through its API. An agent
creating a work package must provide a project, type, subject, and status — not
because a prompt says so, but because the application schema requires it.

Wrapping these applications with MCP server surfaces means agents inherit the
ontology by construction. The MCP tool contract IS the typed interface to a
validated domain model. There is no separate "entity definition" step.

```
┌──────────────────────────────────────────────────┐
│  Enterprise OSS Application                      │
│  (OpenProject, Twenty CRM, InvenTree, Atlas)     │
│           ↕ Postgres schema                      │
│  Application API (validated, typed)              │
│           ↕                                      │
│  MCP Server (typed tool surface)                 │
│           ↕                                      │
│  Agent (inherits ontology via tool contract)      │
└──────────────────────────────────────────────────┘
```

This is not "integration." This is ontology inheritance. The agent does not
need a separate knowledge base about what a work package is — the tool contract
defines it.

The key property: these schemas evolve upstream. When OpenProject adds a new
custom field type, the MCP server exposes it automatically. When Atlas CMMS
adds meter reading validation, agents inherit that validation. The ontology
maintenance cost to PuranOS is near zero for these sources — the open-source
communities bear it.

Self-hosting is load-bearing here. SaaS equivalents (Monday.com, Salesforce,
ServiceNow) offer APIs, but the schemas are opaque, rate-limited, and subject
to vendor changes without notice. Self-hosted Postgres schemas are inspectable,
stable, and under operator control.


### Source 2: Shared Pydantic contract schemas

Industrial engineering has its own ontology that no off-the-shelf software
provides. PuranOS defines it explicitly through shared Pydantic models that
generate 16 JSON Schema files with canonical `$id` URIs. This source has expanded
beyond classical engineering schemas to cover instrumentation, compliance,
project controls, and inter-system exchange contracts.

**Stream-state model (formerly plant-state).** A 31-component process stream model
based on the mASM2d (modified Activated Sludge Model 2d) component basis. Each
stream carries flow, temperature, pressure, pH, individual component
concentrations, and computed aggregates. Stream state is Postgres-primary, stored
in the `stream_snapshot` table via engineering-mcp tools; filesystem JSON export
is available for local tooling but is not the durable store. Multiple component
bases are supported (mASM2d, MCAS for WaterTAP solutes, mADM1 for 62-component
anaerobic models) with typed converters carrying provenance between them.

**Process-unit taxonomy.** Approximately 100 unit process types organized
across 8 treatment areas, aligned with an ISA-95-inspired hierarchy (Area →
Work Center → Work Unit). Each unit type defines its function,
treatment area, and expected equipment list.

**Equipment and component tagging.** Equipment items carry ISA 5.1-compliant
tag structures (e.g., 340-B-01A for Area 340, Blower, Unit 01, Instance A).
Tags include sub-equipment components (motors, VFDs, bearings, seals) with
their own typed attributes.

**Instrumentation and control.** ISA 5.1 loop schemas with function group
encoding. Loops, instruments, control groups, and signal types are typed
entities, not free text annotations on a drawing.

**Artifact envelope.** Every engineering deliverable (equipment list, P&ID,
control philosophy, cost estimate) is wrapped in a typed envelope that declares
its type, version, dependencies on other artifacts, and conformance claims
against declared standards.

**Model credibility.** Every simulation result carries explicit credibility
metadata:

- Model status: validated, calibrated, heuristic, preliminary, stub
- Decision grade: design, budgetary, screening, order-of-magnitude
- Validation basis: bench-tested, plant-data, literature, vendor, assumed

This matters because an agent-generated sizing is not automatically
"engineering grade." The credibility metadata is what allows a PE reviewer to
know what level of trust to place in a result. Credibility metadata now also
includes IDTA AAS "Provision of Simulation Models" fields: license,
environment, parameterization method, and simulation purpose.

**Beyond the original six.** The engineering schema set has expanded to 16
generated schemas covering alarm management (ISA-18.2), cause-effect matrices
(ISA-5.2), instrument databases (ISA-5.1/DEXPI), hydraulic profiles, control
execution (ISA-88/IEC 61131-3), HAZOP studies (IEC 61882), water compliance
(NPDES), project controls (ANSI/EIA-748), and five Ensaras inter-system
exchange contracts. Equipment identity now includes OPC UA DI and IDTA AAS
nameplate fields for industry-standard equipment data exchange.


### Source 3: Custom domain schemas in Postgres

Some business functions require schemas that neither off-the-shelf OSS nor
engineering standards fully cover.

**Procurement.** A 12-entity relational model linking designators (equipment
type codes with aliases) to vendors, quotes, equipment specifications, and cost
observations. Cost observations are append-only (bitemporal — both observation
time and effective time), linked to CEPCI cost indices and calibration factors
for parametric scaling. Project estimates aggregate estimate lines that
reference designator-level cost data and carry AACE estimate class metadata.
Scope basis covers: purchased, installed, package, bare module, TIC, and
service.

**Bid specification review.** Structured evidence chains from document
ingestion through extraction to review. Field catalogs define what to look for
by equipment family. Jobs track bid documents through a defined lifecycle
(draft → ingested → extracting → reviewing → complete). Sections map document
structure (CSI codes, page ranges). Requirements are extracted with raw and
normalized values, modality classification, and review status. Risk flags are
generated with severity levels. Cross-job comparison views enable systematic
evaluation across competing bids.

**Compliance.** Pydantic-validated regulatory calculation models (e.g., LCFS
CA-GREET pathway calculations) with monthly operational data inputs and
auditable calculation chains.


---


## Why this beats the alternatives

| Approach | Ontology Source | Maintenance | Agent Integration | Failure Mode |
|---|---|---|---|---|
| Schema'd tools via MCP | Inherited from tools + purpose-built | Near-zero for OSS; controlled for custom | Native (typed tool calls) | None — ontology IS the tool |
| Generic memory (mem0, etc.) | None (free-form extraction) | High (curation, deduplication) | Bolt-on retrieval | Garbage context accumulation |
| RAG over documents | None (chunked text) | Medium (re-indexing) | Retrieval-only, no write path | Hallucination, staleness |
| Custom knowledge graph | Manual definition | Very high | Custom adapters | Schema drift, maintenance burden |
| Palantir-style unification | Retrofitted over existing systems | Enormous (ongoing reconciliation) | Platform-locked | Multi-year, multi-million-dollar projects |
| Governed identity (PuranOS) | Canonical registry from day one | Near-zero (registry + lifecycle hooks) | Native (same MCP surface as all tools) | None — identity IS the schema |

The critical difference is the maintenance burden. Enterprise OSS schemas are
maintained by their upstream communities. Purpose-built engineering schemas
change only when the engineering domain changes. Custom domain schemas change
only when the business process changes. None of these require ongoing "memory
curation."


---


## Governed identity from day one

The classic industrial data problem: the maintenance team uses one system,
engineering uses another, procurement uses a third, and none of them agree on
what an "equipment item" is. The equipment tag in the CMMS does not match the
line item in the ERP which does not match the asset in the P&ID. Large firms
spend millions on Palantir Foundry, OSIsoft PI, and similar platforms to
retrofit an ontology over systems that were never designed to share one.

PuranOS starts with the ontology. Every enterprise tool is self-hosted
open-source software with an inspectable, forkable Postgres schema. Every
engineering entity has a governed identity from the moment it first appears on
a drawing. The equipment identity registry — a dedicated database and MCP
server surface — implements ISO 14224's functional-position / physical-asset
two-layer model:

- **Functional position**: the slot in the plant (ISA 5.1 tag + UUID).
  Permanent. Survives physical replacements.
- **Physical asset instance**: the thing installed at that slot (CMMS ID +
  serial number). Replaceable. Tracks maintenance history.

When engineering draws P-001 on a P&ID, the position is registered. When
procurement awards a vendor quote, the lifecycle advances. When maintenance
commissions the physical pump, the asset instance links to the position. One
governed master identity connects every system — not through ad-hoc
point-to-point mappings, but through a canonical registry that every MCP server
can resolve.

Self-hosting is load-bearing here. When the CMMS needed a native
`equipment_tag` field on its Asset entity, we forked the open-source codebase,
added the column and a Liquibase migration, and deployed the custom build in
under a minute. With a SaaS vendor, that request enters a feature backlog and
ships on someone else's timeline — if at all. The ability to modify the schema
when the ontology demands it is what makes governed identity practical, not
theoretical.

This is the positive thesis: not "we avoid Palantir's problem" but "we build
the identity infrastructure that makes cross-system resolution a solved problem
from day one." A firm that starts with schema'd tools and governed identity
never accumulates the integration debt that forces incumbents into multi-year
unification projects.


---


## Schema as memory

The three-layer model that replaces a separate "memory system":

```
┌─────────────────────────────────────────────────────┐
│  Layer 1: Session persistence                       │
│  Conversational continuity within a task            │
│  (per email thread, per project context)            │
├─────────────────────────────────────────────────────┤
│  Layer 2: PM tool persistence                       │
│  Task-level state: owner, status, blockers,         │
│  approvals, due dates, comments, predecessors       │
│  (OpenProject — shared between humans and agents)   │
├─────────────────────────────────────────────────────┤
│  Layer 3: Application schemas                       │
│  Domain-level facts: equipment specs, vendor        │
│  quotes, maintenance history, project parameters,   │
│  treatment models, cost data, compliance records    │
│  (Postgres-backed tools — the schema IS the memory) │
└─────────────────────────────────────────────────────┘
```

Together, these three layers provide everything a "memory system" promises —
conversational continuity, task tracking, and domain knowledge — without the
maintenance burden, garbage accumulation, or ontology drift that plague
purpose-built memory layers.

Notice what is absent: there is no vector store, no embedding pipeline, no
deduplication logic, no "memory curation" step, no scheduled re-indexing job.
The data lives in the tools where it was created. Agents query it through
typed interfaces. Humans see exactly the same data through those same tools'
native UIs. There is one source of truth, not a source of truth and a
memory-layer copy of it.

This also eliminates a subtle failure mode: stale memory. In a memory-layer
architecture, the memory can disagree with the tool. A work package marked
complete in OpenProject might still show as "in progress" in the memory store
if the sync job failed. In a schema-as-memory architecture, this class of bug
does not exist.


---


## Research backing

The schema-over-memory thesis is supported by several lines of research:

- **StructMemEval (2026 preprint)** found that structured memory outperforms
  unstructured memory only when organized into task-appropriate structure
  (ledgers, trees, trackers). Free-form memory blobs do not reliably outperform
  simple context windows. The lesson: structure matters, and the best structure
  is one that matches the domain — which is what application schemas provide.

- **"Lost in the Middle"** showed that LLM retrieval accuracy degrades when
  relevant information is embedded in long contexts. More context is not
  reliably better. Schema'd queries (SQL, typed API calls) retrieve exactly
  the relevant data without the noise.

- **StateFlow** demonstrated that explicit state machines improve agent success
  rates by 13-28% at 3-5x lower cost compared to ReAct-style reasoning.
  Schema'd state is a form of explicit state.

- **Agent Workflow Memory** showed that reusable workflows give 24.6-51.1%
  relative improvement over episodic memory. Procedural knowledge (how to do
  something) beats episodic knowledge (what happened last time).

- **Context compression research** shows that pruning and compressing
  conversation history cuts token costs while preserving task accuracy.
  Schema'd queries avoid the compression problem entirely — they return
  structured data, not conversation history.

For detailed treatment: [Schema Over Memory Research](../research/schema-over-memory.md)


---


## What this means in practice

A concrete example: an agent sizing an RO system for a project.

With a memory/RAG approach, the agent would search for "similar past RO
projects" in a vector store, retrieve chunks of text about previous designs,
and attempt to extract relevant parameters. The result depends on what was
stored, how it was chunked, and whether the retrieval finds the right context.

With PuranOS's schema approach, the agent:

1. Queries the stream-state schema for the current stream composition
2. Calls the WaterTAP engine with typed parameters to run the RO simulation
3. Gets back results with explicit model credibility metadata
4. Queries the procurement schema for vendor quotes on RO membranes
5. Produces an artifact envelope with typed dependencies and conformance claims

Every step operates on schema'd data. There is no retrieval ambiguity, no
garbage context, and no question about what the data means.

The difference compounds over time. After 50 projects, the memory/RAG system
has 50 projects worth of unstructured chunks to search through, with growing
retrieval noise and deduplication problems. The schema'd system has 50 projects
worth of typed records in Postgres, queryable with exact filters on project ID,
equipment type, membrane area, recovery rate, or any other schema'd field.

The schema approach also provides a natural audit trail. Every query is a
typed API call with known parameters and deterministic results. There is no
question about "what context did the agent see" — the tool call log is the
complete record.


---


## Implications for agent design

Schema-over-memory has direct consequences for how agents are built:

**Tool selection replaces retrieval.** Instead of deciding what to retrieve
from a memory store, agents decide which tool to call. Tool selection is a
well-understood problem with strong priors from the tool's input schema. Memory
retrieval is an open-ended similarity search with unpredictable results.

**Error handling is typed.** When a schema'd tool call fails, the error is
structured: missing required field, invalid enum value, foreign key constraint
violation. When a memory retrieval fails, the failure mode is silence — the
relevant memory simply was not retrieved, and the agent proceeds with
incomplete context.

**Composition is natural.** An agent can chain tool calls: query the plant
state, pass the result to the simulation engine, pass the simulation output to
the cost estimator. Each handoff is typed. In a memory architecture, composing
retrieval results requires the agent to parse, interpret, and re-structure
free-form text between steps.

**Testing is deterministic.** Schema'd tool calls can be tested with standard
unit and integration tests. Memory retrieval is inherently probabilistic —
the same query may return different results depending on the embedding model,
the chunking strategy, and the contents of the store.


---


## Further reading

- [Coordination Substrate](coordination-substrate.md) — how OpenProject serves as the shared board
- [Ontology Layers](../architecture/ontology-layers/README.md) — detailed map of all three ontology sources
- [Schema Over Memory Research](../research/schema-over-memory.md) — detailed research evidence
