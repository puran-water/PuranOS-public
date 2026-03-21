# Ontology Layers

## Why this exists

The ontology of a wastewater design-build-operate firm spans engineering, commercial, operational, and regulatory domains. No single tool covers it all. No single standard defines it. No single database can contain it.

PuranOS assembles its ontology from three source types, each with different ownership, update cadence, and schema governance. Understanding these sources — and why each exists — is essential to understanding how the system achieves schema'd state without a monolithic data model.

This document maps the full ontology landscape across all three sources. It is an architectural view: what entities exist, where they live, and why.

---

## The three ontology sources

| Source | Governance | Update Cadence | Schema Authority |
|---|---|---|---|
| Enterprise OSS schemas | Upstream open-source projects | Community release cycles | Application Postgres schema |
| Purpose-built engineering schemas | PuranOS engineering team | As-needed, versioned | Pydantic models + JSON Schema |
| Custom domain schemas | PuranOS domain teams | Per-business-function | Pydantic models + migrations |

Each source addresses a different failure mode:
- Enterprise OSS prevents reinventing project management, CRM, inventory, and maintenance.
- Engineering schemas prevent the loss of process engineering semantics that no off-the-shelf tool carries.
- Custom domain schemas prevent the contortion of general-purpose tools into domain-specific roles they were not designed for.

---

## Source 1: Enterprise OSS schemas

Self-hosted open-source applications are the first ontology source. Each application brings a Postgres-backed domain model that was designed, refined, and maintained by an upstream community.

| Application | Domain | Key Entities | Why This Tool |
|---|---|---|---|
| OpenProject | Project management | Work packages, projects, types, statuses, priorities, custom fields, relations, users, roles, memberships | Rich predecessor/successor semantics, distinct assignee/accountable roles, open-source Postgres-backed |
| Twenty CRM | Customer relationships | Contacts, companies, opportunities, activities, pipelines, notes, tasks | Modern CRM with clean API, self-hosted, Postgres-backed |
| InvenTree | Parts and inventory | Parts, stock items, BOMs, suppliers, purchase orders, manufacturers, categories, parameters | Purpose-built for manufacturing/engineering parts management |
| Atlas CMMS | Maintenance | Assets, locations, work orders, preventive maintenance schedules, meters, parts inventory, vendors | CMMS designed for physical asset-intensive operations |
| NocoDB | Structured tables | User-defined tables, views, forms, automations | Flexible structured data for domains not covered by purpose-built tools |

The key insight: these tools were chosen for their schemas, not their UIs. The UI is a convenience. The schema is the value. Wrapping each application with an MCP server surface means agents inherit a structured, validated, Postgres-backed domain model. The API contract enforces the ontology.

```
┌──────────────────────────────────────────────────────────┐
│  Enterprise OSS Ontology Layer                           │
│                                                          │
│  OpenProject    Twenty CRM    InvenTree    Atlas CMMS    │
│  (work)         (commercial)  (parts/BOM)  (assets)      │
│       │              │             │            │        │
│       └──────────────┴─────────────┴────────────┘        │
│                       |                                   │
│            MCP Server Surfaces                            │
│            (typed tool contracts)                          │
│                       |                                   │
│            Agent inherits ontology                         │
│            by construction                                │
└──────────────────────────────────────────────────────────┘
```

Each MCP server exposes a curated subset of the application's full schema. The server acts as a typed interface: it constrains what agents can do, validates inputs against the application's domain model, and returns structured outputs. The agent never touches the database directly. It speaks the application's ontology through a contract-enforced surface.

---

## Source 2: Purpose-built engineering schemas

Industrial process engineering has its own ontology that no off-the-shelf software provides. A CRM does not know what a process stream is. A PM tool does not know what model credibility means. An inventory system does not distinguish between a centrifugal pump and a positive displacement pump at the process engineering level.

PuranOS defines this ontology explicitly through shared Pydantic contract schemas. These live in `libs/engineering-utils/` as Pydantic models; JSON Schema mirrors are generated with canonical `$id` URIs at `https://puranwater.com/schemas/`. Source 2 has expanded from classical engineering schemas to cover instrumentation, compliance, project controls, and inter-system exchange contracts — 16 generated schemas from 12 model modules.

### Stream state (formerly plant-state)

The stream-state schema is the central engineering data model. It represents process streams as multi-component vectors on standardized component bases. Stream state is Postgres-primary, stored in the `stream_snapshot` table via engineering-mcp tools. Filesystem JSON export is available for local tooling but is not the durable store. The `StreamState` Pydantic model supersedes the earlier `plant-state.schema.yaml`; the "plant-state" name is retained as a familiar alias.

Each stream carries:
- **Physical properties**: flow (m3/d), temperature (K), pressure (kPa), pH
- **Component concentrations**: individual components on a declared basis (e.g., 31 components on mASM2d)
- **Computed aggregates**: COD variants, BOD5, TSS, VSS, TKN, TN, TP, alkalinity, TDS, conductivity, ionic strength
- **Metadata**: source model, unit ID, artifact ID, timestamp, version

Multiple component bases are supported:

| Basis | Components | Domain |
|---|---|---|
| mASM2d | 31 | Biological treatment (activated sludge, BNR) |
| MCAS | ~30 solutes | Membrane/desalination (WaterTAP) |
| mADM1 | 62 | Anaerobic digestion |

Typed converters carry provenance when translating between bases. A stream converted from mASM2d to mADM1 retains a record of the source basis, conversion method, and any assumptions applied.

### Process-unit taxonomy

Approximately 100 unit process types organized across 8 treatment areas, following an ISA-95-inspired hierarchy:

```
Area
 └── Unit
      └── Equipment Module
           └── Control Module
```

Each unit type defines its treatment function, expected equipment, and position in the treatment train. The taxonomy is the backbone of the process flow diagram and the equipment list.

### Equipment and components

Equipment items carry ISA 5.1-compliant tag structures following the area-code-sequence pattern:

```
340-B-01A
 |  | |  |
 |  | |  └── Instance (A = first of parallel pair)
 |  | └───── Sequence number
 |  └─────── Equipment type code (B = Blower)
 └────────── Area code (340 = Aeration)
```

The equipment-item schema defines:
- **Identity**: `equipment_uid` (UUID), ISA 5.1 tag, equipment type code, process unit type
- **Nameplate** (OPC UA DI / IDTA AAS): manufacturer, model, serial number, product code, device class, hardware/software revision
- **Sizing**: capacity (value + unit), driver power, sizing parameters
- **Materials**: wetted material, construction material, lining
- **Costing**: optional costing metadata

### Equipment identity: functional position vs physical asset

Equipment identity has two layers, following ISO 14224 and ISO 55000:

**Functional position** — the slot in the plant. Named by the ISA 5.1 tag (e.g., P-001), identified by an immutable UUID (`equipment_uid`). When P-001's pump is replaced, the position stays; the physical asset changes. Positions carry lifecycle stage (design → procurement → construction → commissioning → operations → decommissioned) and cross-system links to the DEXPI model, OpenProject work package, and procurement datasheet.

**Physical asset instance** — the thing installed at that position. Tracked by CMMS asset ID and serial number. When the pump fails and is replaced, the old instance is decommissioned, a new one is commissioned at the same position.

This separation resolves a common industrial data problem: the P&ID tag, the CMMS asset ID, the procurement quote, and the ERP line item all reference different aspects of the same equipment. The tag names the position. The CMMS ID tracks the physical thing. The UUID is the governed master identity that connects them.

```
Functional Position (equipment_uid)
├── current_tag: "P-001"          ← ISA 5.1 (mutable via rename with history)
├── lifecycle_stage: "operations"  ← advances through project phases
├── dexpi_model_id                 ← engineering-mcp P&ID model
├── openproject_wp_id              ← construction/commissioning task
└── Current Asset Instance
    ├── cmms_asset_id: 42          ← Atlas CMMS maintenance record
    ├── serial_number: "SN-12345"  ← nameplate
    ├── manufacturer: "Rashi"
    └── vendor_quote_id: 17        ← procurement origin
```

The identity registry is a dedicated database with its own MCP server surface (`equipment-identity-mcp`), not embedded in any single application's schema. This is intentional: equipment identity spans every domain — design, procurement, construction, operations, disposition — and no single application should own it. The registry is the governed translation layer. The anti-pattern is not having a translation layer; it is having ad-hoc, uncontrolled point-to-point mappings scattered across systems.

### Instrumentation and control

ISA 5.1 loop schemas define instrumentation:
- Loops with function group encoding and measured variable identification
- Instruments with ISA letter codes, range, accuracy, and output signal type
- Control groups linking related loops and instruments
- Signal type classification (analog input, digital output, calculated, etc.)

### Artifact envelope

Every engineering deliverable is wrapped in a typed envelope:
- Artifact type (equipment list, P&ID, control philosophy, cost estimate)
- Version with timestamp
- Dependencies on other artifacts (an equipment list depends on sizing results)
- Conformance claims against declared standards (DEXPI, ISA 5.1)
- Standards conformance claims

The envelope is what makes deliverables machine-readable and dependency-trackable. Without it, a PDF is just a file. With it, a PDF is a versioned artifact in a dependency graph.

### Model credibility

Every simulation result carries credibility metadata:

| Field | Values | Purpose |
|---|---|---|
| Model status | validated, calibrated, heuristic, preliminary, stub | How mature is the model |
| Decision grade | design, budgetary, screening, order-of-magnitude | What decisions can this support |
| Validation basis | bench-tested, plant-data, literature, vendor, assumed | What evidence backs the model |

Credibility metadata now also includes IDTA AAS "Provision of Simulation Models" fields: license, environment, file format, parameterization method, and simulation purpose.

Credibility metadata prevents a common failure mode: using a screening-grade model to make a design-grade decision. The metadata travels with the result, not in a separate document.

### Alarm definitions

Rationalized alarm database per ISA-18.2. Each alarm carries tag, setpoint, units, priority, probable cause, consequence, operator response, deadbands, delays, and rationalization metadata. Stored row-level in Postgres for efficient queries.

### Cause-effect matrices

ISA-5.2 / IEC 62424 cause-effect matrix representation. Sparse matrix with cause and effect lists, linked by cells carrying matrix marks (X, T, D, A, S). Persisted as a single engineering artifact.

### Instrument database

ISA-5.1 instrument index and IO list. Structures loops, instruments, device specifications, measurement ranges, IO signals, and hazardous area classifications per IEC 62424. Includes off-page connectors and source P&ID references.

### Hydraulic profile

Node-by-node hydraulic grade line (HGL) and energy grade line (EGL) across a treatment train. Auto-validates gravity flow feasibility and identifies nodes requiring pumping. Unit-bearing field names: `design_flow_m3_d`, `tailwater_level_m`, `hgl_m`, `egl_m`.

### Control execution

ISA-88 / ISA-95 / IEC 61131-3 control execution hierarchy: process segments, control loops, interlocks (with SIL levels and bypass/reset semantics), control sequences with steps, PLC program organization units, and task bindings.

### HAZOP and LOPA

IEC 61882 HAZOP study vocabulary: nodes, deviations (guideword + parameter), causes, consequences, safeguards, and actions. Includes LOPA (Layer of Protection Analysis) scenarios with IPL credit factors and SIL ratings per IEC 61511.

### Water compliance

NPDES permit and DMR (Discharge Monitoring Report) vocabulary per 40 CFR 122/123. Facilities, discharge points, permit conditions, monitoring results, exceedance events, and regulatory submissions. Tracks compliance status and source systems (SCADA, LIMS, manual, OCR, NetDMR).

### Project controls

ANSI/EIA-748 earned value management: baselines, EV snapshots (PV, EV, AC, CPI, SPI, TCPI, EAC, ETC, VAC), cash flow forecasts, cost-to-complete projections, and contingency drawdowns by phase.

### Ensaras exchange contracts

Five data exchange contracts for the PuranOS → Ensaras operations handover:

| Contract | Standards | Cadence |
|---|---|---|
| Commissioning handover package | IDTA AAS, ISA-95, ISO 14224 | One-time at commissioning |
| Process performance summary | ISA-95 | Monthly |
| Maintenance history exchange | ISO 14224 | Monthly |
| Compliance status exchange | NPDES/DMR | Monthly |
| Capex improvement request | BCF-XML | Event-driven |

### Schema generation pipeline

Pydantic models in `libs/engineering-utils/` are the source of truth. JSON Schema files are generated artifacts with canonical `$id` URIs at `https://puranwater.com/schemas/{filename}`. This makes schemas machine-linkable and version-controlled from a single Python source.

| Group | Schema | Standards Alignment | What It Defines |
|---|---|---|---|
| Core | Stream-state | OPC UA AnalogItem, IDTA AAS Time Series, ISO 15926 | 31-component process stream with flow, T, P, pH, aggregates |
| Core | Equipment-item | OPC UA DI, IDTA AAS, ISA-95, ISO 14224, CFIHOS | Equipment identity, nameplate, sizing, materials, costing |
| Core | Model-credibility | IDTA AAS Simulation Models | Model status, decision grade, validation basis, license |
| Instrumentation | Alarm-definition | ISA-18.2 | Rationalized alarms: tag, setpoint, priority, deadbands |
| Instrumentation | Cause-effect-matrix | ISA-5.2, IEC 62424 | Sparse C&E matrix with standard mark types |
| Instrumentation | Instrument-database | ISA-5.1, DEXPI, IEC 62424 | Instrument index + IO list: loops, instruments, signals |
| Engineering | Hydraulic-profile | Engineering practice | Node-by-node HGL/EGL with gravity feasibility check |
| Engineering | Control-execution-package | ISA-88, ISA-95, IEC 61131-3 | Process segments, control loops, interlocks, PLC programs |
| Engineering | Hazop-study-package | IEC 61882, IEC 61511 | HAZOP nodes/deviations with LOPA scenarios |
| Compliance | Water-compliance-package | NPDES, 40 CFR 122/123 | Facilities, permits, monitoring results, exceedances |
| Project Controls | Project-controls-package | ANSI/EIA-748 | EV baselines, snapshots, cash flow, contingency |
| Ensaras Exchange | Commissioning-handover-package | IDTA AAS, ISA-95, ISO 14224 | Equipment registry, spare parts, baselines, documents |
| Ensaras Exchange | Process-performance-summary | ISA-95 | Monthly operational metrics per equipment |
| Ensaras Exchange | Maintenance-history-exchange | ISO 14224 | Completed work orders from CMMS |
| Ensaras Exchange | Compliance-status-exchange | NPDES/DMR | Permit compliance status per discharge point |
| Ensaras Exchange | Capex-improvement-request | BCF-XML | Improvement requests with viewpoints and cost estimates |

Retained YAML schemas (not yet generated from Pydantic):

| Schema | Standards | What It Defines |
|---|---|---|
| Tag-format | ISA 5.1 | Area-code-sequence-suffix pattern validation |
| Loop | ISA 5.1 | Instrumentation loops with function group encoding |
| Component-tag | ISA 5.1 | Sub-equipment components with typed attributes |
| Valve-schedule | API 6D, IEC 62424 | Valve types, sizes, actuators, failure positions |
| Line-list | PIP PIC001, ISO 10628 | Piping line designations with service, material, size |
| Artifact-envelope | Custom | Typed deliverables with version, dependencies, conformance claims |
| Process-unit-taxonomy | ISA-95 | ~110 unit types across 8 treatment areas |

### System ownership and bridge keys

PuranOS splits data ownership across six systems, each with exclusive write access:

| System | Owns | Bridge Key |
|---|---|---|
| PostgreSQL (3 databases) | Engineering artifacts, stream snapshots, alarms, cost observations, datasheets, equipment registry | `equipment_uid` (UUID) |
| OpenProject | Workflow: tasks, punch items, ITPs, commissioning, RFIs, submittals | `openproject_wp_id` |
| Atlas CMMS | Maintenance execution, PM schedules, failure tracking | `cmms_asset_id` |
| InvenTree | Parts, inventory, BOMs | `inventree_part_id` |
| Twenty CRM | Counterparties: vendors, clients, contractors | `vendor_quote_id` |
| Google Drive | File storage (2 TB) | document URLs |

`equipment_uid` (UUID) is the canonical equipment identity key. It uniquely identifies a functional position (the slot in the plant). OpenProject linkage is indirect — the equipment record carries an `openproject_wp_id` field, not the other way around. Cross-system joins happen at the application level, not via cross-database foreign keys.

For detailed treatment of the engines that produce and consume these schemas, see [Engineering Engines](../../approach/engineering-engines.md).

---

## Source 3: Custom domain schemas

Some business functions require schemas that neither off-the-shelf OSS nor engineering standards fully cover. These are purpose-built for the firm's specific workflows.

### Procurement

A 12-entity relational model covering equipment costing and vendor management:

```
designator ───> vendor_quote ───> cost_observation
(equipment       (pricing,         (append-only
 type codes)      scope,            bitemporal
                  logistics)        cost events)
     |                                   |
equipment_spec                    cepci_index
(detailed                        (cost escalation
 specifications)                  factors)
     |                                   |
project_estimate ───> estimate_line ───> calibration_factor
(AACE class,         (per-item          (parametric
 project scope)       cost lines)        adjustments)
```

Key design choices:
- Cost observations are append-only with bitemporal tracking (both when observed and when effective). Cost history is never overwritten.
- CEPCI indices enable cost escalation across years.
- Scope basis distinguishes purchased, installed, package, bare module, TIC, and service costs.
- Designator-model crosswalk links procurement categories to simulation model identifiers.

### Bid specification review

Structured evidence chains for systematic bid evaluation:
- Field catalogs define extraction targets by equipment family.
- Jobs track bid documents through a defined lifecycle (draft, ingested, extracting, reviewing, complete).
- Sections map document structure using CSI codes and detection tiers (bookmark, heading, regex).
- Requirements are extracted with raw and normalized values, modality classification (shall, should, may), and review status.
- Risk flags carry severity levels (critical, high, medium, low, info).
- Cross-job comparison views enable systematic evaluation across competing bids.

### Compliance

Pydantic-validated regulatory calculation models with monthly operational data inputs and auditable calculation chains. Includes LCFS CA-GREET pathway calculations for biogas-to-RNG compliance, with every intermediate value preserved for regulatory audit.

Water permit compliance (NPDES/DMR) and project controls (earned value, cash flow) now have dedicated Pydantic contracts under Source 2 — they crossed the threshold from custom domain logic into shared, standards-aligned schemas suitable for cross-tool consumption.

---

## The narrow-waist pattern

All three sources converge on a canonical set of shared schemas that define the entities every tool and agent must understand.

```
┌──────────────────────────────────────────────────────────┐
│  Source 1:          Source 2:          Source 3:          │
│  Enterprise OSS     Shared Pydantic    Custom Domain     │
│  (OpenProject,      Contracts          Schemas           │
│   CRM, InvenTree,   (16 generated      (procurement,     │
│   Atlas CMMS)       schemas)           bid review)       │
│         |                |                  |            │
│         +----------------+------------------+            │
│                          |                               │
│              Shared Schemas                              │
│              (the "narrow waist")                        │
│                                                          │
│       Core: stream-state . equipment-item                │
│             model-credibility                            │
│       Instrumentation: alarm . C&E . instruments         │
│       Engineering: hydraulic . control . HAZOP           │
│       Compliance: water-compliance                       │
│       Project Controls: project-controls                 │
│       Exchange: 5 Ensaras handover contracts             │
│       Retained: artifact-envelope . taxonomy . tags      │
│                          |                               │
│              Every tool and agent                        │
│              speaks this entity model                    │
└──────────────────────────────────────────────────────────┘
```

This is the enterprise's narrow waist. Wide diversity of tools and applications above. Wide diversity of storage and implementation below. A common entity model in between.

Any tool that produces an equipment item must produce one that conforms to the equipment-item schema. Any tool that generates a deliverable must wrap it in an artifact envelope. Any simulation result must carry model credibility metadata.

The narrow waist is enforced at the MCP server boundary. Agents never bypass it. The server validates conformance before accepting or emitting data.

---

## How this differs from traditional integration

Traditional enterprise integration connects systems through point-to-point adapters or middleware (ESB, iPaaS). The result: N systems require O(N^2) integrations, each with its own data mapping.

The narrow-waist pattern requires each system to speak one shared entity model. N systems require O(N) adapters — one per system — and any new system only needs to implement the shared schemas to interoperate with every existing system.

| Approach | Integration Cost | Ontology Source | New System Cost |
|---|---|---|---|
| Point-to-point | O(N^2) adapters | Implicit in each adapter | Integrate with every existing system |
| ESB / middleware | O(N) + middleware | Middleware schema (often ad-hoc) | Implement middleware connector |
| Narrow waist (PuranOS) | O(N) MCP servers | Shared schemas (explicit, versioned) | Implement shared schema conformance |

The narrow-waist approach has a second advantage beyond integration cost. Because the shared schemas are explicit and versioned, they serve as documentation. A new engineer reading the equipment-item schema learns what the firm considers an equipment item. A new agent reading the plant-state schema learns what the firm considers a process stream. The ontology is self-documenting.

---

## Schema governance

Shared schemas follow a simple governance model:

1. **Pydantic models** are the source of truth for engineering entity definitions. They define types, constraints, and validation logic in code. Currently 16 contract schemas from 12 model modules.
2. **JSON Schema mirrors** are generated from the Pydantic models via `generate_schemas.py`. Each carries `$id: "https://puranwater.com/schemas/{filename}"` as a canonical logical identifier.
3. **Breaking changes** require a version bump and migration. Fields can be added without a version bump. Fields cannot be removed or renamed without one.
4. **Conformance tests** verify that MCP server outputs match the shared schemas. These run in CI.

This governance model keeps the ontology coherent as the system grows. Adding a new MCP server does not require coordinating with every existing server. It requires conforming to the shared schemas.

---

## Entity cross-reference

Where each major entity type originates and where it flows:

| Entity | Origin Source | Produced By | Consumed By |
|---|---|---|---|
| Work package | Enterprise OSS (OpenProject) | Project management agents | All agents (task coordination) |
| Equipment position | Equipment identity registry | PE lead (at P&ID digitization) | All agents (cross-system resolution) |
| Asset instance | Equipment identity registry | Maintenance (at commissioning) | CMMS, procurement, inventory |
| Equipment item | Engineering schema | Engine MCP servers (sizing) | Procurement, costing, P&ID generation |
| Stream state (formerly plant-state) | Engineering schema | Engine MCP servers (simulation), Postgres-primary via stream_snapshot | Downstream engines, reports, deliverable generation |
| Cost observation | Custom domain (procurement) | Procurement agents | Estimating, bid evaluation |
| Process datasheet | Custom domain (procurement) | Procurement agents, O&M manual ingestion | Maintenance, operations |
| Artifact envelope | Engineering schema | Any deliverable-producing tool | Version tracking, dependency resolution |
| Model credibility | Engineering schema | Engine MCP servers | Decision-support agents |
| BOM entry | Enterprise OSS (InvenTree) | Equipment agents | Procurement, inventory |
| Work order | Enterprise OSS (Atlas CMMS) | Maintenance agents | Operations, scheduling |
| Alarm definition | Engineering schema | Control philosophy workflows | SCADA config, alarm rationalization, operations |
| Cause-effect matrix | Engineering schema | Control philosophy workflows | Instrument wiring, SIS design |
| Hydraulic profile | Engineering schema | Hydraulic design workflows | Civil/structural, pump sizing |
| Control execution | Engineering schema | Control design workflows | PLC programming, commissioning |
| HAZOP study | Engineering schema | Safety review workflows | SIL assessment, action tracking |
| Water compliance | Engineering schema | Compliance agents | NPDES reporting, exceedance tracking |
| Project controls | Engineering schema | PM analytics | EV reporting, cash flow forecasting |
| Compliance record | Custom domain (compliance) | Compliance agents | Regulatory reporting |

---

## Standards alignment

The engineering schemas in Source 2 are aligned with industry consensus standards. The alignment is intentional: when an industry standard defines an entity well, the PuranOS schema follows it rather than inventing a proprietary alternative.

For the full mapping between PuranOS schemas and industry standards, see [Standards and Conformance](../../approach/standards-and-conformance.md).

---

## Further reading

- [Schema Over Memory](../../approach/schema-over-memory.md) -- why schema-first approaches outperform memory and RAG
- [Standards and Conformance](../../approach/standards-and-conformance.md) -- how schemas map to industry standards
- [Engineering Engines](../../approach/engineering-engines.md) -- the engines that produce and consume these schemas
- [MCP Servers](../mcp-servers/) -- the server surfaces that enforce ontology conformance
