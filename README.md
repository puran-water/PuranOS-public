# PuranOS-public

PuranOS-public is the documentation-only companion to PuranOS, Puran Water LLC's integrated operating system for industrial engineering, project delivery, procurement, compliance, and governed agent execution.

It is built for Puran Water LLC and for industrial design-build firms working in industrial wastewater, treatment, recovery, and waste-to-value infrastructure. The platform is tuned for that operating environment today, while the architectural patterns remain extensible to other industrial engineering enterprises.

## The Central Thesis

Building an AI-native industrial firm requires a properly schema'd data substrate — not a memory or RAG system bolted on later. That substrate comes from three sources: self-hosted enterprise OSS (OpenProject, CRM, inventory, CMMS — each brings a Postgres-backed domain ontology), purpose-built engineering schemas (31-component plant-state model, ~110 process unit types, ISA 5.1 instrumentation, DEXPI equipment classes, model credibility metadata), and custom domain schemas (procurement, bid specification review, compliance). All exposed via typed MCP tool surfaces.

This avoids the "Palantir-style" rework where firms with ad-hoc systems must later unify their ontology. The ontology exists by construction — it is the schema of the tools themselves.

## Why This Repo Exists

Most industrial firms do not suffer from a lack of software. They suffer from fragmented execution:

- engineering models live in one toolchain
- project work lives in another
- commercial follow-through lives elsewhere
- automation ends up trapped in brittle scripts or disconnected assistants

PuranOS is designed to close that gap. It combines engineering-grade computation, durable orchestration, shared project state, and controlled tool execution into one operating model.

This public repo documents that architecture without exposing private infrastructure, credentials, or internal deployment details.

## What PuranOS Does

### Engineering and Technical Delivery

- process engineering workflows for aerobic and anaerobic treatment, RO, IX, evaporation, degassing, corrosion, hydraulics, and plant-state reasoning
- multi-interface engineering engines built around QSDsan and WaterTAP
- shared engineering utilities for converters, heuristics, and model handoff
- reusable skills that turn specialist engineering workflows into repeatable operating procedures

### Project Execution and Agent Collaboration

- OpenProject as the shared board for long-running work, dependencies, approvals, and human-agent collaboration
- a communication and orchestration runtime that receives inbound work, routes it to the correct persona, and persists execution state durably
- MCP servers that give agents typed, governed access to project, engineering, finance, and operations systems

### Commercial, Operations, and Back Office

- CRM, procurement, inventory, CMMS, accounting, compliance, and reporting integrations
- specialist personas for project management, engineering, sales, procurement, maintenance, legal, and finance
- tool-level governance so side effects are enforced where the tools execute, not only in prompts

## Architecture At A Glance

```text
External systems and human operators
            ↓
Communication / orchestration runtime
            ↓
Persona layer + reusable skills
            ↓
MCP server layer
            ↓
Business systems + engineering engines
```

The most important design choice is the hybrid state model:

- OpenProject stores long-running collaborative project state
- PostgreSQL stores execution reliability state such as deduplication, attempts, transitions, leases, and side-effect audit records

That split keeps project work legible to humans without sacrificing the exact semantics required for reliable agent execution.

## Start Here

**Understand the approach:**
- [Why PuranOS Is Built This Way](docs/approach/README.md) — the five architectural bets
- [Schema Over Memory](docs/approach/schema-over-memory.md) — the flagship thesis

**See the architecture:**
- [Architecture Overview](docs/architecture/README.md) — layers, hybrid state model, MCP contracts
- [Persona Boundaries](docs/architecture/personas/README.md) — why scoped agents matter
- [Ontology Layers](docs/architecture/ontology-layers/README.md) — the three sources of schema'd state

**Read the evidence:**
- [Research Index](docs/research/README.md) — coordination, memory, and counter-evidence

**Learn more:**
- [For Curious Industrial Engineers](docs/community/README.md) — who this is for and how to connect

## Documentation Map

### Approach and Design Philosophy

- [Why PuranOS Is Built This Way](docs/approach/README.md)
- [The Primacy of Schema'd State](docs/approach/schema-over-memory.md)
- [OpenProject as Coordination Substrate](docs/approach/coordination-substrate.md)
- [Why Skills Compound](docs/approach/skills-as-expertise.md)
- [First-Class Engineering Computation](docs/approach/engineering-engines.md)
- [Industrial Standards Alignment](docs/approach/standards-and-conformance.md)

### Architecture

- [Architecture Overview](docs/architecture/README.md)
- [Agent Runtime](docs/architecture/agent-runtime/README.md)
- [MCP Servers](docs/architecture/mcp-servers/README.md)
- [Persona Boundaries](docs/architecture/personas/README.md)
- [Ontology Layers](docs/architecture/ontology-layers/README.md)

### Research and Evidence

- [Research Index](docs/research/README.md)
- [Coordination and State](docs/research/coordination-and-state.md)
- [Schema Over Memory](docs/research/schema-over-memory.md)

### Operations

- [Provisioning Guide](docs/provisioning/README.md)
- [Communication Agent](docs/runtime/communication-agent/README.md)
- [OpenProject MCP](docs/servers/openproject-mcp.md)
- [OpenProject Service Notes](docs/services/openproject.md)

### Community

- [For Curious Industrial Engineers](docs/community/README.md)
- [The Skill Model](docs/community/skill-contribution-guide.md)

## Public Repo Scope

This repository is intentionally documentation-only. It is meant to explain:

- the operating model
- the architectural decisions
- the collaboration patterns
- the deployment principles

It is not a public mirror of the full internal monorepo.

## Source Snapshot

This documentation export reflects the internal PuranOS monorepo at source commit `889f982`.

## Contact

For technical collaboration:

- Puran Water LLC
- Hersh Kshetry, Founder & Principal Engineer
- Website: [puranwater.com](https://puranwater.com/)
- Contact page: [puranwater.com/contact](https://puranwater.com/contact/)
- Schedule a call: [calendar.app.google/M1jzSdCB51sYiWux6](https://calendar.app.google/M1jzSdCB51sYiWux6)
