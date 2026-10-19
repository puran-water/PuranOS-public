# Architecture overview

PuranOS organizes engineering around a retained project basis and usable handoffs. It supports Puran Water’s process engineering, equipment, project-economics and operating work.

## Layers and responsibilities

| Layer | Responsibility |
| --- | --- |
| Source evidence | Design requirements, measurements, literature, vendor information and owner decisions, with their authority and revision. |
| Specialist agents | Prepare bounded work using the tools and skills exposed to their role. |
| Engineering tools | Perform defined calculations, compile/check geometry, prepare schedules and render deliverables. |
| Domain records | Retain equipment identity, process information, procurement requirements, financial assumptions and related artifacts. |
| Coordination and runtime | Track tasks, dependencies, attempts, policy and recovery. |
| Review and handoff | Make results inspectable, record decisions and issue the agreed documents/data to the next team. |

## Follow one equipment decision

1. Process work establishes the duty and its source basis.
2. The equipment record supplies stable identity; a displayed tag can change without inventing a new physical role.
3. CAD refers to that identity when placing equipment and connections. Access and clearance checks use a defined model and registry basis.
4. Deliverable workflows read retained records to produce schedules and document packages.
5. Review identifies missing information and determines the release status.
6. The customer receives agreed human-readable files and structured information or authorized access.

This is a workflow pattern, not a claim that every system synchronizes automatically. Each integration must define the fields, revision behavior and writer responsible for a change.

## State has different owners

OpenProject holds collaborative project work. PostgreSQL-backed runtime records retain attempts and execution state; Hatchet supports durable workflow execution where configured. Engineering and commercial domains own their data and mutations. An ontology/read layer can traverse declared relationships without becoming an unrestricted writer into every system.

Documents and drawings are outputs with identity and status. Their presence does not replace the source records or prove approval. Knowledge articles preserve narrative rationale and lessons alongside structured fields.

## Trust and access

Persona instructions describe responsibilities. Available MCP tools, runtime policy, server checks and deployment permissions determine actual authority. A configured tool is not evidence that a particular project grants permission to use it.

The public MCP endpoint exposes service-discovery tools. Private customer access must define project scope, authentication, permitted operations and revision semantics. See [integration guidance](../integration/deliverables.md).

## Related guides

[Specialists](personas/README.md) · [MCP](mcp-servers/README.md) · [Identity](ontology-layers/README.md) · [Runtime](agent-runtime/README.md) · [Current state](../current-state.md)
