# MCP Servers

PuranOS uses MCP as the stable interface between agents and the systems they operate. In this repo, that interface is tuned for the kinds of toolchains an industrial wastewater and waste-to-value design-build firm actually needs: simulation engines, project controls, procurement, operations systems, and commercial follow-through.

## Why MCP Matters Here

This repo spans many domains, many runtimes, and many applications. Without a stable tool contract, every agent prompt would need to know too much about every API.

MCP solves that by centralizing:

- authentication and API quirks
- typed inputs and outputs
- governance and side-effect boundaries
- reusable integration logic

That keeps the personas simpler and makes the repo easier to evolve.

## Current Server Families

### Project Delivery and Collaboration

| Server | Role |
|--------|------|
| `openproject-mcp` | OpenProject work packages, comments, relations, views, custom fields, and typed agent-task collaboration |
| `cpm-mcp` | Schedule extraction, network analysis, what-if analysis, and OpenProject writeback for scheduling workflows |
| `pm-analytics` | Project health, earned value, risk scan, design-build status, and portfolio-style project rollups |
| `twenty-mcp` | CRM operations for revenue and pipeline workflows |

### Operations, Inventory, and Execution

| Server | Role |
|--------|------|
| `atlas-cmms-mcp` | CMMS interactions for maintenance operations |
| `inventree-mcp` | Inventory and manufacturing-adjacent operations |
| `procurement-mcp` | Procurement workflow support |
| `quickbooks-mcp` and `wave-mcp` | Accounting and finance system integration |

### Engineering and Technical Computation

The `processeng-workspace` contains the bulk of the domain-specific engineering surface. It includes server families for:

- treatment process design
- diagramming and layout
- corrosion, heat transfer, and fluids analysis
- WaterTAP- and QSDsan-oriented development work
- discipline-specific design servers such as degassers, evaporators, RO, IX, and more

Plant state (process stream data) is managed via filesystem JSON files and the `plant-state-skill`, not as an MCP server. Stream state files at `mcp-outputs/streams/` conform to the plant-state schema and are validated by engineering-utils Pydantic models.

This is supported by shared utilities in `libs/engineering-utils/`.

Two worktree themes are especially important now:

- QSDsan has become a multi-interface process engine with stronger public registries, mixed-model conversions, and downstream WaterTAP handoff.
- WaterTAP has shifted toward a shared action-catalog architecture with session persistence, registry-backed discovery, detached jobs, and stronger MCP/CLI parity.

At the monorepo level, that means the engineering stack is converging on reusable orchestration engines rather than isolated solver wrappers.

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

This repo tries to keep each layer focused:

- MCP servers define tool contracts and integration behavior
- personas define who can use the tools and under what constraints
- skills define how work should be carried out once tools are available

That separation is especially valuable in a monorepo with both computation-heavy engineering servers and business-operations integrations.

## Customization

When adding a new server:

1. define stable tool names and clear typed inputs
2. document mutation boundaries explicitly
3. add only the personas that truly need access
4. update the relevant architecture docs

Avoid pushing domain logic into persona prompts when it belongs in a server contract or a reusable skill.
