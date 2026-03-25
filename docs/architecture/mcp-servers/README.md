# MCP Servers

PuranOS uses MCP as the stable interface between agents and the systems they operate. In this repo, that interface is tuned for the kinds of toolchains an industrial wastewater and waste-to-value design-build firm actually needs: simulation engines, project controls, procurement, operations systems, and commercial follow-through.

## Why MCP Matters Here

This repo spans many domains, many runtimes, and many applications. Without a stable tool contract, every agent prompt would need to know too much about every API.

MCP solves that by centralizing authentication, typed contracts, governance boundaries, and reusable integration logic. But MCP is more than a tool-calling protocol — it defines four primitives that each serve a distinct purpose:

| Primitive | Direction | Purpose | PuranOS usage |
|-----------|-----------|---------|---------------|
| **Tools** | Agent → Server | Execute actions with typed inputs/outputs | 200+ tools across 21 servers: simulations, CRUD, governed mutations |
| **Resources** | Server → Agent | Expose read-only data the agent can pull on demand | `ontology://dossier/equipment/{uid}`, `cmms://workorders/open`, `openproject://project/{id}/summary`, `kb://collections` |
| **Prompts** | Server → Agent | Structured prompt templates with embedded context | `explore_entity` (ontology neighbor graph + lifecycle + actions), `lifecycle_audit` (state distribution + validation hints) |
| **Elicitation** | Server → User | Request structured human input during tool execution | Destructive operation confirmation gates on Atlas CMMS (delete work orders, assets, PM schedules) |

**Why this matters:** Tools alone make MCP a function-calling protocol. Resources make it a context protocol — agents can pull live system state without executing a tool call. Prompts embed domain expertise into the server contract so agents don't need it in their persona instructions. Elicitation creates human-in-the-loop checkpoints for irreversible operations without requiring the agent to implement confirmation logic.

The practical effect: a maintenance persona can read `cmms://workorders/open` as ambient context, execute `work_order(operation="change_status")` as a governed action, and the server autonomously gates destructive deletions through `_elicit_delete_confirmation()` — all within a single MCP session. The persona prompt never mentions any of this. The server contract owns the behavior.

## Current Server Families

### Project Delivery and Collaboration

| Server | Role |
|--------|------|
| `openproject-mcp` | OpenProject work packages, comments, relations, views, custom fields, and typed agent-task collaboration. Resources: project summaries, milestones, assigned tasks, custom field schemas. |
| `cpm-mcp` | Schedule extraction, network analysis, what-if analysis, and OpenProject writeback for scheduling workflows |
| `pm-analytics` | Project health, earned value, risk scan, design-build status, and portfolio-style project rollups |
| `twenty-mcp` | CRM operations for revenue and pipeline workflows |

### Operations, Inventory, and Execution

| Server | Role |
|--------|------|
| `atlas-cmms-mcp` | CMMS interactions for maintenance operations. Resources: asset summaries, open work orders, upcoming PM, low-stock parts. Elicitation gates on destructive operations (delete WO/asset/PM). |
| `inventree-mcp` | Inventory and manufacturing-adjacent operations (parts, stock, BOMs, purchase orders, barcodes, labels) |
| `procurement-mcp` | Equipment procurement: vendor quotes, cost observations, datasheets, estimates, vendor guarantees. Write-only for domain data; reads via ontology-mcp. |
| `quickbooks-mcp` and `wave-mcp` | Accounting and finance system integration |
| `contractor-management-mcp` | DBIA contractor/subcontractor commercial management: contracts, SOV lines, G702/G703 pay applications, change order detail, warranty/LD claims. Thin financial sidecar to OpenProject. |

### Engineering and Technical Computation

The `processeng-workspace` contains the bulk of the domain-specific engineering surface. It includes server families for:

- treatment process design
- diagramming and layout
- corrosion, heat transfer, and fluids analysis
- WaterTAP- and QSDsan-oriented development work
- discipline-specific design servers such as degassers, evaporators, RO, IX, and more

Stream state (process stream data) is Postgres-primary, stored in the `stream_snapshot` table via engineering-mcp tools. Filesystem JSON export is available for local tooling but is not the durable store. The `plant-state-skill` provides the workflow SOP for stream reasoning.

This is supported by shared utilities in `libs/engineering-utils/`, which provides 23 contract schemas as Pydantic models with generated JSON Schema mirrors, plus 98 Postgres table schemas introspected from the live databases.

Two worktree themes are especially important now:

- QSDsan has become a multi-interface process engine with stronger public registries, mixed-model conversions, and downstream WaterTAP handoff.
- WaterTAP has shifted toward a shared action-catalog architecture with session persistence, registry-backed discovery, detached jobs, and stronger MCP/CLI parity.

At the monorepo level, that means the engineering stack is converging on reusable orchestration engines rather than isolated solver wrappers.

### Cross-Domain Ontology

| Server | Role |
|--------|------|
| `ontology-mcp` | Universal read layer across all 6 Postgres databases. Tools: `get_object`, `list_objects`, `resolve_display_fields`, `find_related`, `validate_action`, `list_allowed_actions`, `classify_equipment`, `rename_project`. Resources: equipment dossier (single traversal across all DBs), project summary, neighbor graph. Prompts: `explore_entity`, `lifecycle_audit`. 198 typed links, 19 governed actions, 21 lifecycle state machines. |

### Compliance, Finance, and Growth

Additional workspaces support:

- compliance calculations and reporting
- lead generation and external opportunity discovery
- project finance workflows
- knowledge and retrieval

## OpenProject as a First-Class Collaboration Server

`openproject-mcp` deserves special attention because it is central to the repo's agentic collaboration model.

Its current role is not just CRUD. It exposes the operations needed for collaborative agent work:

- standard project and work package operations
- comments, relations, hierarchy, and views
- custom-field and schema helpers
- typed agent-task creation and retrieval
- explicit agent state updates
- agent result posting and completion flows

This is what allows the communication-agent, project-management persona, and human team members to collaborate on the same shared board.

## Boundary Discipline

This repo keeps each layer focused:

- **MCP servers** define the full contract: tools (actions), resources (context), prompts (domain expertise), and elicitation (human checkpoints)
- **Personas** define who can access which servers and under what constraints — they do not implement tool behavior
- **Skills** define how work should be carried out once tools are available — procedural knowledge, not integration logic

That separation is especially valuable because:
- Resources let agents pull ambient context without the persona needing to know the query shape
- Elicitation lets servers enforce human-in-the-loop gates without the agent implementing confirmation flows
- Prompts embed domain expertise (e.g., "how to explore an ontology entity") in the server, not scattered across persona instructions

## Customization

When adding a new server:

1. Define stable tool names with clear typed inputs and `annotations` (readOnlyHint, destructiveHint, idempotentHint)
2. Add resources for any data agents should be able to pull as ambient context
3. Add elicitation gates for destructive or irreversible operations
4. Document mutation boundaries explicitly
5. Add only the personas that truly need access
6. Update the relevant architecture docs and ontology action manifests

Avoid pushing domain logic into persona prompts when it belongs in a server contract or a reusable skill.
