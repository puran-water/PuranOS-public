# Why PuranOS Is Built This Way

## Why this exists

Industrial firms don't lack software. They lack a shared operating model
connecting engineering computation, project execution, commercial
follow-through, and operational memory.

The result: every project reinvents workflows, knowledge evaporates between
projects, and AI integration attempts fail because there is no schema for
the AI to operate on.

Most firms attempting AI adoption start with chatbot wrappers or document
retrieval systems. These fail at scale because they have no structured
understanding of the business domain. There is no schema for equipment. No
ontology for process units. No typed relationship between a vendor quote and
the equipment it prices.

PuranOS takes a different path. Instead of bolting AI onto existing ad-hoc
systems, it builds the schema'd substrate first and makes AI a native
consumer of that substrate.


## The five architectural bets

### 1. Schema'd state over memory

A properly schema'd database substrate is the primary knowledge store,
reducing the need for separate AI "memory" systems. The ontology comes from three sources:

| Source | Examples | What it provides |
|--------|----------|-----------------|
| Enterprise OSS schemas | OpenProject, CRM, inventory, CMMS | Postgres-backed domain ontology per vertical |
| Shared Pydantic contract schemas | 16 generated schemas: stream-state, equipment-item (OPC UA DI/AAS), model-credibility, alarm definitions (ISA-18.2), cause-effect matrices (ISA-5.2), instrument databases, hydraulic profiles, control execution (ISA-88), HAZOP (IEC 61882), water compliance (NPDES), project controls (EIA-748), 5 Ensaras exchange contracts | Typed engineering objects with known relationships and standards-aligned vocabulary |
| Custom domain schemas | Procurement (12-entity relational model), bid spec review (structured evidence chains), compliance (validated regulatory calculations) | Domain logic that no off-the-shelf tool covers |

All exposed via typed MCP tool surfaces. An agent does not need to
"remember" what equipment exists on a project. It queries the schema'd
state and gets back typed objects with known relationships.

Link: [Schema Over Memory](schema-over-memory.md) — the flagship document
explaining this thesis in detail.


### 2. PM tool as coordination substrate

A project management system (OpenProject) serves as the shared board for
human and AI bidirectional task delegation.

Research supports this choice. Explicit per-task state beats prompt memory.
Moving from scratchpad-based state to explicit, externalized state took
target recovery from 0.213 to 1.000 in multi-agent coordination benchmarks.

The key insight: the PM tool is not a reporting dashboard. It is the
runtime coordination layer. Agents read their assigned work packages,
execute against typed tool surfaces, and write results back to the work
package. Humans review, approve, and assign new work through the same
interface.

Link: [Coordination Substrate](coordination-substrate.md)


### 3. Skills as captured expertise

Reusable operating procedures encode how work should be done, not just what
tools to call.

Skills compound value over time. Each successful project produces workflows
that can be encoded as skills for future projects. An experienced process
engineer describing "how I size an MLE bioreactor" produces a more valuable
skill than a software developer writing Python — because the engineer
encodes decision logic, not just computation.

Skills are not prompt templates. They are structured operating procedures
with typed inputs, typed outputs, decision trees, and references to the
specific MCP tools required at each step.

Link: [Skills as Expertise](skills-as-expertise.md)


### 4. Engineering engines as first-class citizens

Deterministic simulation engines are infrastructure, not afterthoughts.

| Engine | Domain | Role |
|--------|--------|------|
| QSDsan | Biological and chemical treatment | Mass/energy balance, process simulation, LCA |
| WaterTAP | Membrane and separation processes | Desalination, advanced treatment, costing |

Both engines provide:

- Session persistence across agent interactions
- Cross-engine handoffs via typed converters (e.g., QSDsan effluent
  becomes WaterTAP influent with unit conversion and component mapping)
- Model credibility metadata on every result, so downstream consumers
  know whether a number came from a screening estimate or a calibrated
  model

Link: [Engineering Engines](engineering-engines.md)


### 5. Tool-level governance

Side-effect permissions are enforced at the MCP server boundary, not in
prompts.

When an agent sends an email, the server enforces which identity it sends
from. When an agent creates a work package, the server validates it has the
required fields. When an agent modifies a vendor record, the action is
logged with the agent identity and the tool call that triggered it.

Prompt-level governance ("please don't do bad things") is not governance.
Server-boundary enforcement with audit trails is governance.

These actions are auditable after the fact. The system can answer "which
agent modified this vendor record, when, and in response to what task" from
structured logs.


## What this is not

| Claim | Why it does not apply |
|-------|----------------------|
| Chatbot wrapper | There is no "ask PuranOS" interface. Agents execute structured workflows against typed tools. |
| Palantir-style data unification | The ontology is not retrofitted over existing ad-hoc systems. It is the schema of the tools themselves, from day one. |
| SaaS product | PuranOS is self-hosted infrastructure for a specific operating firm, designed so that other industrial firms with similar needs can adopt the architecture. |
| "Just add AI" to existing tools | The tools, schemas, and operating model are co-designed. The AI is a native participant, not an afterthought. |


## How the pieces connect

```
┌─────────────────────────────────────────────────────────────────┐
│  The Approach (Why)                                             │
│  Schema'd state · Coordination substrate · Skills · Engines    │
│  Standards alignment · Tool governance                          │
├─────────────────────────────────────────────────────────────────┤
│  The Architecture (How)                                         │
│  Persona boundaries · Ontology layers · Hybrid state model      │
│  MCP server contracts · Orchestration runtime                   │
├─────────────────────────────────────────────────────────────────┤
│  The Evidence (Research backing)                                │
│  Agent coordination research · Schema vs memory literature      │
│  Counter-evidence and how we respond                            │
└─────────────────────────────────────────────────────────────────┘
```

The approach layer (this document) explains the "why" — the architectural
bets and the problems they solve. The architecture layer explains the
"how" — persona boundaries, ontology design, MCP server contracts, and the
orchestration runtime. The evidence layer provides the research backing,
including counter-evidence and how the design responds to it.

Each layer can be read independently. The approach layer is useful without
knowing any implementation details. The architecture layer is useful
without agreeing with every bet. The evidence layer is useful as a
literature review even outside the PuranOS context.


## Standards alignment

PuranOS does not invent ontologies where industry consensus already exists.

| Domain | Standard | How PuranOS aligns |
|--------|----------|-------------------|
| P&ID data exchange | DEXPI (ISO 15926 profile) | Equipment classes, piping, and instrumentation use DEXPI enumerations directly |
| Equipment hierarchy | ISA-95 / IEC 62264 | Area → Unit → Equipment Module → Control Module |
| Instrumentation | ISA 5.1, ISA-5.2, IEC 62424 | Tag format, function groups, instrument letter codes, cause-effect matrices |
| Alarm management | ISA-18.2 | Rationalized alarm definitions with priority, deadband, and rationalization fields |
| Batch/sequence control | ISA-88, IEC 61131-3 | Process segments, control loops, interlocks, PLC programs |
| Safety studies | IEC 61882, IEC 61511 | HAZOP nodes/deviations, LOPA scenarios with SIL ratings |
| Capital handover | CFIHOS | Equipment attributes, nameplate data, materials of construction |
| Device identity | OPC UA DI, IDTA AAS | Equipment nameplate, simulation model provisioning metadata |
| Regulatory reporting | NPDES, 40 CFR 122/123 | Permit conditions, monitoring results, DMR field vocabulary |
| Earned value | ANSI/EIA-748 | PV, EV, AC, CPI, SPI, cash flow forecasts, contingency drawdowns |

Where no standard exists (e.g., model credibility metadata, procurement
ontology, agent task delegation), PuranOS defines its own schemas and
documents the design rationale.

Link: [Standards and Conformance](standards-and-conformance.md)


## Reading guide

The recommended reading order depends on your interest.

**Understand the thesis:**

1. [Schema Over Memory](schema-over-memory.md) — the central argument
2. [Coordination Substrate](coordination-substrate.md) — why OpenProject,
   backed by research

**Understand the operating model:**

3. [Skills as Expertise](skills-as-expertise.md) — how institutional
   knowledge becomes machine-executable
4. [Engineering Engines](engineering-engines.md) — how engineering
   computation is treated as infrastructure
5. [Standards and Conformance](standards-and-conformance.md) — how schemas
   map to industry consensus

**Understand the architecture:**

- [Architecture Overview](../architecture/README.md)
- [Persona Boundaries](../architecture/personas/README.md)
- [Ontology Layers](../architecture/ontology-layers/README.md)

**Read the evidence:**

- [Research Index](../research/README.md)
- [Coordination and State Research](../research/coordination-and-state.md)
- [Schema Over Memory Research](../research/schema-over-memory.md)

**Connect:**

- [For Curious Industrial Engineers](../community/README.md)
- [The Skill Model](../community/skill-contribution-guide.md)
