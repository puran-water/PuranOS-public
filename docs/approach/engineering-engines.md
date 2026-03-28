# First-Class Engineering Computation

## Why this exists

In most engineering firms, simulation models live in Jupyter notebooks,
Excel spreadsheets, or vendor software with no API. They are disconnected
from project state, unrepeatable across projects, and unauditable. A sizing
calculation done six months ago cannot be reproduced because the notebook
was modified, the spreadsheet was overwritten, or the vendor software
version changed.

When an engineer leaves a firm, the institutional knowledge encoded in
those notebooks and spreadsheets leaves with them. When a project is
revisited two years later for an expansion, the original design basis
is effectively lost.

PuranOS treats engineering computation as infrastructure:
session-persistent, cross-engine, credibility-tagged, and
schema-validated. Engineering models are not tools that agents happen
to use. They are first-class citizens of the operating model.


## API-first engineering computation

PuranOS engineering servers are API-first: every calculation is exposed as a typed MCP tool call, callable by agents or scripts. Results are versioned, schema-validated, and carry credibility metadata.

| Server | Capabilities |
|---|---|
| qsdsan-engine-mcp | Biological and chemical simulation — IWA ASM1, ASM2d, mADM1 — activated sludge, anaerobic digestion, clarification, solids handling |
| watertap-engine-mcp | Membrane modeling and techno-economic analysis — RO, NF, ED, IX, evaporation, crystallization — with session-persistent flowsheets |
| water-chemistry-mcp | Aqueous equilibrium, speciation, precipitation, and dosing optimization via PHREEQC (phreeqc.dat, minteq.dat, llnl.dat, pitzer.dat) |
| fluids-mcp | Pipe sizing, pump TDH, control valve sizing (IEC 60534), blower power — 120+ fluid properties via CoolProp |
| heat-transfer-mcp | Shell-and-tube exchangers, surface heat loss, insulation design, heat trace — 390+ material conductivities (VDI/ASHRAE) |
| corrosion-engineering-mcp | Sweet/sour corrosion prediction (deWaard-Milliams), galvanic corrosion, pitting risk, material selection |
| engineering-mcp | P&ID generation (DEXPI equipment classes), BFD/PFD generation (SFILES notation), ISA 5.1 tagging, equipment tag management |

Cross-engine handoffs use typed converters with provenance tracking. A QSDsan effluent composition feeds directly into a WaterTAP membrane model with unit conversion and component mapping. Every intermediate result is schema-validated and carries its credibility grade.


## The operating model

Two simulation engines handle the core engineering domains. Each exposes
both MCP tool surfaces (for agent access) and CLI interfaces (for direct
engineering use).


### QSDsan — Biological and chemical treatment

QSDsan handles the biological and chemical heart of wastewater treatment.
It covers activated sludge processes, anaerobic digestion, clarification,
chemical treatment, and solids handling.

The engine supports four model families with increasing mechanistic
fidelity:

- **Heuristic models** — empirical correlations and design rules of thumb.
- **Steady-state mass balance** — conservation-based sizing.
- **Dynamic simulation** — time-varying behavior (ASM-family kinetics).
- **Mechanistic models** — first-principles process chemistry.

Each model family produces results at a different credibility level. A
heuristic sizing is useful for screening. A validated dynamic simulation
is useful for detailed design. The credibility metadata travels with
the result.


### WaterTAP — Membrane, separation, and costing

WaterTAP handles membrane processes (reverse osmosis, nanofiltration),
thermal separation (evaporation, crystallization), ion exchange, and
integrated costing.

The engine maintains session-persistent flowsheet state. An agent can
build up a treatment train incrementally — add an RO stage, check the
permeate quality, add a second pass, adjust recovery — without losing
the flowsheet between tool calls.

WaterTAP sessions carry their own state: the current flowsheet topology,
unit parameters, stream compositions, and solve status. This is not
conversation memory. It is model state persisted in the engine.


### Shared utilities

A shared engineering library provides the bridge layer between engines:

- **Converters** between component bases (mASM2d, MCAS, mADM1) with
  provenance tracking.
- **Deterministic engineering calculations** — hydraulics, heat transfer,
  unit conversions.
- **Shared Pydantic models** (EquipmentItem, ModelCredibility) used by
  both engines and by skills.
- **Schema definitions** consumed by engineering MCP tools and shared
  skill schemas.

The shared library is deterministic. It contains no LLM calls, no
stochastic behavior, and no network dependencies. Given the same inputs,
it produces the same outputs every time.


## The model handoff problem

Process engineering inherently chains models. A typical treatment train
might flow:

```
Influent characterization
        |
        v
Biological treatment (QSDsan -- mASM2d basis, 31 components)
        |
        v
Solids handling (QSDsan -- anaerobic digestion, mADM1 basis)
        |
        v
Membrane separation (WaterTAP -- MCAS basis)
        |
        v
Brine treatment (WaterTAP -- evaporation/crystallization)
        |
        v
Costing (WaterTAP -- parametric and vendor-based)
```

Each stage may use a different simulation engine with a different
component basis. The 31-component mASM2d model used for activated sludge
does not share a component list with the MCAS solute model used for RO
simulation.

Without typed converters, this handoff is manual. An engineer exports
results from one tool, transforms them in a spreadsheet, and imports
them into the next tool. This is error-prone, unrepeatable, and loses
provenance.

PuranOS solves this with typed converters that:

- Accept a stream in one component basis and produce a stream in another.
- Carry provenance: which converter was used, which version, what
  assumptions were applied.
- Validate compatibility at the boundary. A stream missing required
  components is rejected, not silently passed through.
- Preserve aggregate properties (COD, TSS, TN, TP) as cross-checks.

```
+--------------+    typed      +--------------+    typed      +--------------+
|   QSDsan     |--converter-->| engineering  |--converter-->|   WaterTAP   |
|  (mASM2d)    |              |    utils     |              |   (MCAS)     |
|              |<--converter--|              |<--converter--|              |
+--------------+              +--------------+              +--------------+
```

The converters are bidirectional. A WaterTAP membrane result can feed
back into a QSDsan model for recycle stream calculations. Each direction
has its own converter with its own assumptions and limitations
documented in the provenance record.


## Model credibility metadata

Every simulation result in PuranOS carries explicit credibility metadata.
This is not optional. It is part of the result schema.


### The three credibility dimensions

**Model status** — how well the model has been verified:

| Status | Meaning |
|---|---|
| Validated | Compared against independent data; performance confirmed |
| Calibrated | Fitted to site-specific or project-specific data |
| Heuristic | Based on empirical correlations and design rules |
| Preliminary | Initial estimate; parameters not yet confirmed |
| Stub | Placeholder; no real calculation behind the number |

**Decision grade** — what level of decision the result supports:

| Grade | Meaning | Typical Use |
|---|---|---|
| Design | Suitable for detailed design and PE sign-off | Final engineering |
| Budgetary | Suitable for budget estimates and planning | Preliminary engineering |
| Screening | Suitable for technology comparison and feasibility | Concept studies |
| Order-of-magnitude | Rough estimate within a factor of ~3-5x | Early-stage scoping |

**Validation basis** — what evidence supports the model parameters:

| Basis | Meaning |
|---|---|
| Bench-tested | Parameters from bench-scale or pilot testing |
| Plant-data | Parameters from full-scale plant operating data |
| Literature | Parameters from published research |
| Vendor | Parameters from vendor-supplied performance data |
| Assumed | Parameters assumed from typical ranges |


### Why this matters

Without credibility metadata, all simulation results look the same. A
preliminary heuristic sizing and a validated dynamic simulation both
produce a number. They have fundamentally different reliability.

In traditional practice, this context lives in the engineer's head. They
know that "this sizing is based on vendor data from a similar plant" or
"this is just a textbook correlation." That context disappears when the
model is handed to someone else, reused on a different project, or
referenced months later.

Model credibility metadata makes the context explicit and
machine-readable. An agent generating a cost estimate can check whether
the upstream sizing is "validated + design grade" or "heuristic +
screening grade" and adjust its approach accordingly. A PE reviewer can
see at a glance what level of trust to place in each result.

The credibility schema is defined as a shared Pydantic model and JSON Schema. Several servers already attach credibility metadata to their outputs. The architectural intent is universal enforcement at the tool surface — a simulation result without credibility metadata should fail schema validation. This is partially implemented today and being extended across all engine outputs.


## Session persistence

Both engines maintain persistent sessions. This means:

- An agent can build up a model incrementally over multiple tool calls
  without losing state.
- Intermediate results are available for inspection and modification.
- Sessions can be resumed after interruption.
- The model state at any point can be captured and reproduced.

This is fundamentally different from stateless API calls where each
request must contain the complete model specification. Session
persistence allows the natural engineering workflow — iterative
refinement — to work with agents the same way it works with human
engineers.

A typical engineering session involves dozens of adjustments. Change a
design flow rate. Re-solve. Check the effluent quality. Adjust an
aeration setpoint. Re-solve. Compare. This iterative loop is how
process engineers work. Stateless tools force the engineer to rebuild
the entire model for each iteration. Session-persistent engines
preserve the working state.


## Why this matters for design-builders

Engineering models that carry their own credibility metadata, feed into
downstream models via typed converters, and persist results in project
state are fundamentally different from "ask an LLM to size a pump."

The difference:

| Aspect | LLM-Only Approach | PuranOS Engine Approach |
|---|---|---|
| Calculation basis | Parametric knowledge from training data | Deterministic simulation with explicit models |
| Reproducibility | Non-deterministic | Deterministic (same inputs, same outputs) |
| Credibility | Unknown — "the AI said so" | Explicit metadata on every result |
| Cross-model handoff | Manual copy-paste or hope | Typed converters with provenance |
| Auditability | Conversation log | Schema-validated results with artifact envelopes |
| PE sign-off | Difficult — no verifiable basis | Tractable — credibility and provenance are explicit |

An agent-generated sizing is not automatically "engineering grade." The
engines, credibility metadata, and typed handoffs are what make
agent-assisted engineering auditable and trustworthy.

Design-build firms carry professional liability for their designs. The
engineer of record signs and seals drawings. That engineer needs to
understand the basis of every calculation. Credibility metadata and
provenance tracking exist to serve that need — not to add bureaucracy,
but to make the computational basis of a design as transparent as the
hand calculations it builds on.


## Further reading

- [Schema Over Memory](schema-over-memory.md) — the schema-validated
  state foundation these engines build on.
- [Standards and Conformance](standards-and-conformance.md) — how engine
  outputs align with industry standards.
- [Ontology Layers](../architecture/ontology-layers/README.md) — where
  engineering schemas fit in the ontology.
