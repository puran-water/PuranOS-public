# Architecture Overview

PuranOS is intentionally layered. It is built for Puran Water LLC and for industrial design-build firms working in industrial wastewater, treatment, recovery, and waste-to-value infrastructure. That focus shapes the engineering stack and workflow design, while the underlying architecture remains extensible to other industrial engineering businesses.

## Why This Exists

Industrial delivery work spans multiple time horizons:

- a design calculation might take seconds
- a procurement action might span days
- a project task with human predecessors might remain blocked for weeks

If every layer stores state the same way, the system becomes either fragile or opaque. This repo avoids that by giving each layer a clear job.

## Current Architectural Model

### 1. Service Layer

The service layer provides the durable business applications and infrastructure:

- project management
- CRM
- inventory
- CMMS
- shared databases
- ingress and tunnel infrastructure

These services are configured under `services/`.

### 2. MCP Server Layer

The MCP layer is the integration contract between agents and systems. Servers expose typed tools instead of raw API calls, which makes behavior easier to test, document, and govern.

This repo contains server families for:

- project delivery and collaboration
- process engineering and technical design
- commercial operations and CRM
- procurement and inventory
- compliance and reporting
- finance and accounting
- knowledge and retrieval

Two especially important engineering engines now follow the same broad pattern:

- QSDsan as a multi-interface biological/process simulation engine
- WaterTAP as a session-persistent orchestration engine for treatment and costing workflows

Both increasingly rely on `libs/engineering-utils/` for shared bridge logic, converters, and downstream handoff semantics. That library now provides 16 contract schemas (from 12 model modules) covering engineering, instrumentation, compliance, project controls, and inter-system exchange — generated from Pydantic models and published as JSON Schema with canonical URIs at `puranwater.com/schemas/`.

See [MCP Servers](mcp-servers/README.md).

### 3. Persona Layer

Personas are the operating roles. They define:

- what tools are available
- what workflows are expected
- what side effects are allowed
- how work should be framed for a given function

Examples include project management, sales, procurement, maintenance, legal, bookkeeping, and multiple engineering specialties.

Persona discovery is now more dynamic than earlier repo snapshots. The runtime can discover specialist configs from `agent-configs/` and pair them with persona-local skill overlays.

### 4. Skill Layer

Skills encode repeatable work. They make the system more valuable over time because expertise is captured once and reused many times.

### 5. Knowledge Layer

Not all relevant context reduces to typed fields in a schema'd database. Meeting synthesis, design rationale, competitive intelligence, lessons learned, and vendor evaluations are knowledge artifacts that need to be synthesized, linked, and queried — not filed as records or embedded in a vector store.

The Knowledge Wiki is a file-backed markdown vault (Obsidian-compatible format) exposed to agents through the `wiki-graph` MCP server — a fork of mcpvault adding wikilink graph traversal, fragment reads, safe rename-with-link-updates, health tools, and ambient MCP resources on top of the 15 mcpvault file I/O primitives. Drop-flow agents write raw sources into dated channel folders (`raw/meetings/`, `raw/email/`, `raw/teams-chat/`, `raw/whatsapp/`, and others). A dedicated knowledgebase agent owns the reconcile flow: it reads raw, synthesizes compiled articles under `concepts/`, `people/`, `projects/`, `decisions/`, or `glossary/`, maintains the master index, and appends to an append-only operations log. Reconciliation runs nightly; linting runs weekly. The wiki also records phonetic-variant entity resolution (canonical aliases) so ASR-mangled proper nouns don't split the graph.

The default deployment is a single shared `professional` vault on OCI; optional scoped servers (`wiki-project`, `wiki-asset`) can expose materialized read-only corpora for specific projects or assets when hard isolation is needed. Agents are the only readers and writers — humans query the wiki by asking an agent rather than opening the vault directly. The wiki complements Paperless-NGX: documents needing cross-system reference IDs go into Paperless; synthesized knowledge goes into the wiki.

See [Knowledge Wiki](../approach/knowledge-wiki.md) for the full thesis.

### 6. Orchestration Layer

The orchestration layer is where inbound events become controlled execution:

- webhook ingress
- event deduplication
- task dispatch
- retries and leases
- state transitions
- side-effect audit logging
- declarative schedules and prompt-template-driven recurring work

The core implementation lives in `agents/communication-agent/`.

See [Agent Runtime](agent-runtime/README.md).

## The Most Important Design Choice

PuranOS uses a hybrid state model with three substrates:

- OpenProject holds long-running collaborative project state.
- PostgreSQL holds short-lived execution and reliability state.
- The Knowledge Wiki holds synthesized unstructured context — meeting synthesis, design rationale, competitive intelligence, lessons learned — that does not reduce to typed database fields.

That split is not incidental. Schema'd databases handle canonical objects with typed fields and known relationships. The coordination substrate handles task state and delegation. The Knowledge Wiki handles the qualitative context that agents need but that no schema can capture. Each substrate serves a distinct class of concern, and overloading any one of them with the others' responsibilities degrades all of them.

## Architecture Documents

- [Agent Runtime](agent-runtime/README.md) — webhook ingress, orchestration, OpenProject collaboration, policy enforcement
- [MCP Servers](mcp-servers/README.md) — server categories, contracts, and why MCP is the right abstraction here
- [Persona Boundaries](personas/README.md) — why scoped agent roles matter and how delegation works
- [Ontology Layers](ontology-layers/README.md) — the three sources of schema'd state and the narrow-waist pattern
- [Provisioning](../provisioning/README.md) — sanitized deployment model and configuration guidance

## Approach and Research

The architectural choices documented here are backed by specific design philosophy and research:

- [Why PuranOS Is Built This Way](../approach/README.md) — the six architectural bets
- [Schema Over Memory](../approach/schema-over-memory.md) — why schema-first approaches outperform memory/RAG
- [Coordination Substrate](../approach/coordination-substrate.md) — why OpenProject, backed by research
- [Research Index](../research/README.md) — detailed evidence and counter-evidence
