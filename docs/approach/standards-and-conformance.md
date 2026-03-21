# Industrial Standards Alignment

## Why this exists

Purpose-built schemas and OSS tool ontologies must map to industry consensus standards.
Without standards alignment, PuranOS's "forced ontology" argument is incomplete --
you have a schema, but does it mean the same thing the rest of the industry means?

Industrial engineering has well-established standards for data exchange, equipment
identification, process hierarchy, and instrumentation. Ignoring these standards means
building a proprietary ontology that cannot interoperate with clients, vendors,
regulators, or other engineering firms. Aligning with them means the schema'd substrate
speaks the same language as the rest of the industry.

---

## Standards landscape

| Standard | What It Governs | How PuranOS Aligns |
|---|---|---|
| DEXPI (ISO 15926 profile) | P&ID data exchange -- equipment, piping, instrumentation. DEXPI 2.0 uses a standardized DEXPI XML serialization that supersedes the earlier Proteus XML schema. | Engineering MCP uses DEXPI-derived enumerations (re-exported from pyDEXPI, not locally invented). Current alignment is at the enumeration level. Export format targets DEXPI XML (the current standard serialization); legacy Proteus XML export is supported but not the primary format. Full DEXPI conformance validation is a future goal. |
| ISO 15926 | Process plant lifecycle data integration | DEXPI alignment inherits ISO 15926 semantics. Stream-state schema and equipment models use compatible entity identifiers. |
| ISA-95 / IEC 62264 | Enterprise-to-control integration boundaries | Equipment hierarchy is aligned with ISA-95 concepts (Area, Work Center, Work Unit). Process-unit taxonomy uses an ISA-95-inspired hierarchy for organizing ~110 unit types across 8 treatment areas. |
| ISA 5.1 | Instrument identification and tagging | Loop, instrument, and tag schemas encode ISA 5.1 function groups. Tag format schema enforces the area-code-sequence pattern. Instrument types carry ISA 5.1 letter codes. |
| ISO 14224 / ISO 55000 | Equipment reliability and asset management | Equipment identity uses the ISO 14224 functional-position / physical-asset-instance two-layer model. Functional positions (ISA 5.1 tag + UUID) persist across physical replacements. Asset instances carry CMMS ID, serial number, and maintenance history. The equipment identity registry is the governed master that connects P&ID, CMMS, procurement, and project management systems. |
| CFIHOS | Capital facilities handover information | Equipment position and asset instance tables implement core CFIHOS capital handover attributes: functional location, tag, lifecycle stage, manufacturer, model, serial number, commissioning date, and procurement origin. Process datasheets extend coverage with materials of construction, design conditions, and nameplate data. The CFIHOS Starter Class Library (equipment class hierarchy, properties per tag type) is used as reference for designator taxonomy and datasheet field validation. |
| OPC UA DI | Device identity and automation information models | Equipment-item schema includes OPC UA DI nameplate fields (manufacturer, model, serial number, product code, device class, hardware/software revision). Tag and signal typing follows OPC UA variable type semantics. |
| ISA-18.2 | Alarm management lifecycle | Alarm-definition schema uses ISA-18.2 vocabulary: alarm types, priorities, rationalization fields, deadband/delay parameters, probable cause, consequence, and operator response. |
| ISA-5.2 | Binary logic diagrams, cause-and-effect | Cause-effect-matrix schema implements ISA-5.2 sparse matrix representation with standard mark types (X, T, D, A, S). |
| ISA-88 / IEC 61512 | Batch control, recipe/procedure hierarchy | Control-execution-package implements process segments, procedures, and recipe hierarchy for sequential and batch control. |
| IEC 62424 | Instrumentation in process control documentation | Instrument-database schema structures loops, instruments, and IO signals per IEC 62424 document model. Cause-effect matrices follow IEC 62424 representation conventions. |
| IEC 61131-3 | PLC programming languages | Control-execution-package includes POU (Program Organization Unit) types and task binding models for PLC program structure. |
| IEC 61882 | HAZOP study procedure | Hazop-study-package implements the HAZOP vocabulary: nodes, deviations (guideword + parameter), causes, consequences, safeguards, and actions. |
| IEC 61511 | Functional safety (SIS) | LOPA scenarios in the HAZOP schema carry IPL credit factors and SIL ratings per IEC 61511 / CCPS practice. |
| IDTA AAS | Asset Administration Shell (digital nameplate) | Equipment-item includes AAS nameplate fields. Model-credibility includes AAS "Provision of Simulation Models" fields (license, environment, parameterization method). |
| BCF-XML | Issue/defect exchange (buildingSMART) | Capex-improvement-request uses BCF-XML semantics for viewpoints, issue structure, and document references. |
| ANSI/EIA-748 | Earned value management | Project-controls-package implements EV vocabulary: PV, EV, AC, CPI, SPI, TCPI, EAC, ETC, VAC, cash flow forecasts, and contingency drawdowns. |
| NPDES / 40 CFR 122-123 | Water discharge permits | Water-compliance-package uses NPDES/DMR vocabulary for permit conditions, monitoring results, exceedance events, and regulatory submissions. |

---

## Schemas in practice

PuranOS maintains shared schemas that implement practical subsets of these standards
for engineering deliverables. These schemas are the "narrow waist" -- the canonical
entity model that all tools and agents must speak.

### Engineering schemas

#### Generated from Pydantic models (16 schemas)

| Schema | Standards Alignment | What It Defines |
|---|---|---|
| Stream-state (formerly Plant-state) | OPC UA AnalogItem, IDTA AAS Time Series, ISO 15926 | 31-component process stream with flow, T, P, pH, aggregates |
| Equipment-item | OPC UA DI, IDTA AAS, ISA-95, ISO 14224, CFIHOS | Equipment identity, nameplate, sizing, materials, costing |
| Model-credibility | IDTA AAS Simulation Models | Model status, decision grade, validation basis, license, parameterization |
| Alarm-definition | ISA-18.2 | Rationalized alarms: tag, setpoint, priority, deadbands, rationalization |
| Cause-effect-matrix | ISA-5.2, IEC 62424 | Sparse C&E matrix with standard mark types (X, T, D, A, S) |
| Instrument-database | ISA-5.1, DEXPI, IEC 62424 | Instrument index + IO list: loops, instruments, device specs, signals |
| Hydraulic-profile | Engineering practice | Node-by-node HGL/EGL with gravity feasibility validation |
| Control-execution-package | ISA-88, ISA-95, IEC 61131-3 | Process segments, control loops, interlocks, sequences, PLC programs |
| Hazop-study-package | IEC 61882, IEC 61511 | HAZOP nodes/deviations with LOPA scenarios and IPL credit factors |
| Water-compliance-package | NPDES, 40 CFR 122/123 | Facilities, discharge points, permits, monitoring, exceedances |
| Project-controls-package | ANSI/EIA-748 | EV baselines/snapshots, cash flow forecasts, contingency drawdowns |
| Commissioning-handover-package | IDTA AAS, ISA-95, ISO 14224 | One-time handover: equipment registry, spare parts, baselines |
| Process-performance-summary | ISA-95 | Monthly operational metrics per equipment |
| Maintenance-history-exchange | ISO 14224 | Completed work orders from CMMS |
| Compliance-status-exchange | NPDES/DMR | Permit compliance status per discharge point |
| Capex-improvement-request | BCF-XML | Improvement requests with viewpoints and cost estimates |

#### Retained YAML schemas (not yet generated from Pydantic)

| Schema | Standards Alignment | What It Defines |
|---|---|---|
| Component-tag | ISA 5.1 | Sub-equipment components (motors, VFDs, bearings, seals) with typed attributes |
| Tag-format | ISA 5.1 | Area-code-sequence-suffix pattern validation |
| Process-unit-taxonomy | ISA-95 hierarchy | ~110 unit types across 8 treatment areas with function and equipment expectations |
| Loop | ISA 5.1 | Instrumentation loops with function group encoding, instrument lists, control group assignments |
| Line-list | PIP PIC001, ISO 10628 | Piping line designations with service, material, size, rating, and insulation |
| Valve-schedule | API 6D, IEC 62424 | Valve types, sizes, actuators, failure positions, and service conditions |
| Artifact-envelope | Custom | Typed deliverables with version, dependencies, conformance claims, and approval status |

These schemas do not attempt to implement the full scope of any standard. They
implement the subset that matters for the work PuranOS does -- process engineering
design for industrial wastewater treatment. The goal is interoperability with industry
practice, not exhaustive compliance.

### Schema generation and canonical URIs

Pydantic models are the source of truth. JSON Schema files are generated with canonical
`$id` URIs at `https://puranwater.com/schemas/{filename}`. This makes schemas
machine-linkable and version-controlled from a single Python source. Non-Python
consumers (schema validators, documentation generators, exchange partners) use the
generated JSON Schema files.

---

## How standards alignment connects to the ontology

The three ontology sources (see [Ontology Layers](../architecture/ontology-layers/README.md))
each relate to standards differently.

**Enterprise OSS.** These tools bring their own schemas, which do not directly map to
industrial standards. OpenProject's work package model is not ISA-95-aware. But the PM
concepts -- tasks, dependencies, approvals -- are universal across industries. The
alignment need here is structural, not terminological.

**Engineering schemas.** This is where standards alignment matters most. Equipment items,
instrument tags, process unit hierarchies, and P&ID data exchange all have
well-established industry standards. PuranOS's engineering schemas implement these
standards explicitly. When the schema says "centrifugal pump," it means the same thing
DEXPI means by "centrifugal pump."

**Custom domain schemas.** Procurement, bid review, and compliance schemas are
domain-specific. There are no widely adopted standards for wastewater procurement
ontology. These schemas are purpose-built but use standard identifiers (CEPCI indices,
AACE estimate classes, CSI codes) where applicable. They do not invent new numbering
systems when industry-accepted ones exist.

---

## DEXPI alignment: an illustrative example

DEXPI (Data Exchange in the Process Industry) defines how P&ID data should be
structured for exchange between engineering tools. It specifies equipment classes,
piping component types, and instrumentation categories.

DEXPI 2.0 introduced a standardized DEXPI XML serialization that supersedes the
earlier Proteus XML schema. The current DEXPI specification can be used fully
without Proteus XML. PuranOS targets the current DEXPI standard.

PuranOS's engineering MCP server uses DEXPI enumerations directly, re-exported from
the pyDEXPI library. This means:

- Equipment types in PuranOS P&IDs use DEXPI classification codes.
- Piping components use DEXPI-defined types (10 piping enumeration categories).
- Instrumentation uses DEXPI-defined instrument categories.
- The resulting P&ID data targets DEXPI XML export for exchange with other
  DEXPI-compliant tools. Legacy Proteus XML export is also supported.

The enumerations are not locally invented. They are imported from pyDEXPI, which
implements the DEXPI standard. This ensures that PuranOS speaks the same equipment
language as other DEXPI-compliant engineering tools.

The practical implication: P&ID data produced by PuranOS uses the same equipment vocabulary as other DEXPI-aligned tools. Interoperability with specific DEXPI-compliant tools depends on the extent of schema coverage, which is being expanded incrementally.

---

## ISA-95 hierarchy in process-unit taxonomy

The ISA-95 equipment hierarchy provides the structural backbone for how PuranOS
organizes process units. The hierarchy runs:

    Site > Area > Unit > Equipment Module > Control Module

PuranOS's process-unit taxonomy maps onto this hierarchy. Treatment areas (preliminary,
primary, secondary, tertiary, sludge, chemical, utilities) correspond to ISA-95 Areas.
Individual process units (anoxic reactor, membrane bioreactor, gravity thickener)
correspond to ISA-95 Units. Equipment within those units (blowers, pumps, mixers)
corresponds to Equipment Modules.

This is not decorative. The hierarchy determines how tags are structured, how control
loops are scoped, and how equipment rolls up into area-level summaries. It also
determines how PuranOS communicates with SCADA and DCS systems that already use
ISA-95 hierarchy.

---

## Future: conformance assertion

The architecture is designed to support automated validation that artifacts conform to
declared standards. The artifact-envelope schema already includes a field for
conformance claims -- which standards the artifact asserts compliance with.

The goal is that deliverables produced by agent workflows are not free-form AI output.
They are schema-validated engineering artifacts with explicit conformance claims against
declared standards. A P&ID generated through an agent workflow would carry:

- DEXPI conformance claim for the equipment and piping data.
- ISA 5.1 conformance claim for the instrumentation tagging.
- Artifact-envelope metadata with version, dependencies, and approval status.
- Model credibility metadata for any simulation results that informed the design.

This is not fully implemented today. But the schema infrastructure -- artifact
envelopes, conformance fields, credibility metadata -- is in place. The path from
"schema exists" to "conformance is machine-verified" is incremental, not architectural.

### What conformance checking enables

When conformance is machine-checkable, several things become possible:

- **Automated review gates.** A P&ID cannot advance to the next design phase unless it
  passes DEXPI schema validation and ISA 5.1 tag format checks. The gate is not a
  human reviewer eyeballing tag names; it is a schema validator.
- **Interoperability guarantees.** If an artifact claims DEXPI conformance and passes
  validation, it can be exchanged with any other DEXPI-compliant tool without manual
  translation.
- **Audit trails.** Conformance claims are recorded in the artifact envelope. A
  regulator or client can see which standards each deliverable claims to satisfy, and
  whether those claims have been validated.

---

## What this does not cover

Standards alignment is about data semantics and interoperability. It does not address:

- **Regulatory interpretation** -- determining whether a facility is in compliance,
  or what specific permit limits should be. Standards alignment now covers the data
  vocabulary for regulatory reporting (NPDES permit structure, DMR field names via the
  water-compliance-package schema), but compliance determination remains a domain
  logic concern, not a schema concern.
- **Calculation standards** -- Ten States Standards, WEF MOPs, TR-16. These inform
  engineering logic in the simulation engines, not schema structure.
- **Drawing standards** -- ASME Y14.5, company-specific CAD standards. PuranOS
  generates structured data, not drawings. Drawing production from structured data
  is a downstream concern.

The distinction matters. Standards alignment in PuranOS is about ensuring that the
structured data produced by engineering workflows uses the same vocabulary and
hierarchy as the rest of the industry. It is not about ensuring that every calculation
follows every applicable design manual.

---

## Further reading

- [Schema Over Memory](schema-over-memory.md) -- why schema'd state is the foundation
- [Engineering Engines](engineering-engines.md) -- the engines that produce standards-aligned outputs
- [Ontology Layers](../architecture/ontology-layers/README.md) -- how all three ontology sources map to the schema landscape
