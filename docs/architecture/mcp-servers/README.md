# MCP tools and access boundaries

MCP provides a defined interface between an agent and tools or context. In PuranOS, it connects specialist agents to engineering computation and domain systems. Features differ by server and deployment; this guide does not claim a universal SDK version or identical primitive support across the fleet.

## Types of interface

- **Tools** accept defined inputs and return results or perform authorized actions.
- **Resources** expose information for a client to retrieve.
- **Prompts** provide reusable interaction templates.
- **Elicitation**, where implemented, requests additional information or confirmation. It does not itself establish that a client has authority to approve an engineering or commercial action.

## Engineering and business surfaces

| Family | Examples and purpose |
| --- | --- |
| Process computation | WaterTAP, QSDsan, chemistry, fluids and heat-transfer tooling for defined engineering cases. |
| Equipment and engineering records | Equipment-identity and engineering writers; shared process and deliverable artifacts. |
| CAD and engineering analysis | Geometry, registry binding, layout, connections, clearance and drawing workflows. |
| Finance | Proforma assumptions, archived model runs, scenario comparisons and accounting-actuals analysis. |
| Procurement and maintenance | Supplier/equipment evidence, inventory, asset history and maintenance work. |
| Coordination | OpenProject and durable workflow tools for task state and execution. |
| Read and knowledge | Ontology traversal, documents and linked narrative knowledge. |

An internal adapter or tool in source is not a public service entitlement. Determine availability, authorization and project scope for the actual deployment.

## Public customer connection

`https://intake.puranwater.com/mcp` is the public Streamable HTTP endpoint. It has no authentication for its public discovery surface. Its tools describe engagements, check fit and return data-preparation checklists. Optional submission and booking-link tools are separate actions. It does not expose internal simulation, files, financial models or project databases.

See [connection instructions and tool behavior](../../integration/mcp.md).

## Private project integrations

Define the source records and identifiers, schema version, permitted reads/writes, error behavior and review boundary. Prefer the domain’s intended writer for mutations; do not treat an ontology/read connection as blanket write access.

A client can validate a payload’s shape without establishing that its contents are correct or suitable for construction. Access, data validation and engineering release are separate concerns.
