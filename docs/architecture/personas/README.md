# Persona Model

## Why this exists

A single unrestricted agent is ungovernable at enterprise scale.
It can read everything, write anywhere, and make decisions that should
require different expertise or authority levels. The failure mode is not
malice — it is scope creep. An agent asked to size a pump ends up
modifying the project schedule because it has access to everything.

Personas solve this by separating three concerns: who can read what,
who can write where, and which decisions require handoff to a different
role.

This document describes the persona model as an **architectural pattern**.
It is not a catalog of every persona in PuranOS. For the full inventory
see the agent-config manifests in the monorepo.

---

## The pattern

A persona is an agent configuration that binds four scopes:

| Scope       | What it controls                                  | Where it is enforced           |
|-------------|---------------------------------------------------|--------------------------------|
| **Tool**    | Which MCP servers the agent can call               | `.mcp.json` server list        |
| **Expertise** | Which workflows and skills the agent executes    | Agent-config skill references  |
| **Authority** | What side effects the agent may produce          | MCP server-level guards        |
| **Boundary**  | When the agent must delegate to another persona  | Delegation protocol + OpenProject |

Personas are not "AI characters." They are scoped operating contexts —
the same pattern that RBAC applies to human users, extended to agent
execution.

The mental model: onboarding a new hire. You give them a job description
(the persona instructions), system access (the MCP server list), and
SOPs (the skills). A persona is exactly this — except the hire works
continuously and never forgets the SOPs.

```
  persona = tool_scope + expertise_scope + authority_scope + boundary_scope
```

Each scope is independently configurable. Two personas can share a tool
scope but differ in authority scope. This is how read-only consumers
and read-write producers coexist on the same MCP server.

---

## Delegation, not orchestration

Personas do not call each other directly. There is no function call from
pe-process to pe-separation. Instead, a persona that encounters work
outside its scope creates a **work package** in OpenProject, assigned to
the persona with the right scope.

This matters for three reasons:

1. The audit trail records who requested what and who executed it.
2. The receiving persona starts with a clean context window.
3. Human operators can intercept, re-assign, or reject any delegation.

The delegation substrate is OpenProject. See
[Coordination Substrate](../../approach/coordination-substrate.md).

---

## Illustrative chain: the PE team

Rather than cataloging every persona, this section walks through one
complete delegation chain to show how the pattern works.

### pe-lead (orchestrator)

Creates the block flow diagram. Decomposes the design into
discipline-specific tasks. Has access to `engineering-mcp` for BFD
generation, `office-mcp` for email (as `engineering.agent@circleh2o.com`),
and `rlm-bridge` for document review sessions. Creates child work
packages in OpenProject and assigns them to specialist personas.

Stream state data flows between agents via filesystem JSON files at
`{project_dir}/mcp-outputs/streams/`. pe-lead reads these directly.

pe-lead does not size equipment. It frames the problem and delegates.

### pe-process (biological and chemical treatment)

Sizes biological and chemical unit processes using QSDsan and PHREEQC
water chemistry. Reads and writes stream state JSON files for
inter-agent data flow. Cannot modify the project schedule. Cannot
send external communications.

When pe-process needs membrane sizing, it does not attempt it. It
creates a task for pe-separation.

### pe-separation (membrane and thermal separation)

Sizes membrane and thermal separation using WaterTAP. Operates with
session-persistent flowsheets. When it needs upstream water chemistry,
it delegates to pe-process, which owns the water-chemistry tool surface.

### pe-mechanical (fluids, heat transfer, corrosion)

Handles mechanical engineering analysis. Has a critical boundary
discipline: pe-mechanical has **no stream state write access**. Its
results are returned to the calling persona (typically pe-lead) for
state management. This prevents two specialists from writing conflicting
state simultaneously.

### pe-cad (layout and visualization)

Produces site layouts and 3D equipment visualization. Reads stream state
files and equipment geometry but does not modify engineering parameters.

### pe-deliverables (document generation)

Generates equipment lists, I&C schedules, and control philosophy
documents from the current design state. Reads from stream state files
and engineering results, producing typed artifact envelopes.

### cost-estimator (parametric costing)

Produces parametric cost estimates linked to equipment items. Reads
the procurement schema for vendor quote data and the equipment registry
for sizing results.

### Data flow

```
                          pe-lead
                       (orchestrator)
                      /   |    |    \
                     /    |    |     \
              pe-process  |  pe-mech  pe-cad
                  |    pe-sep  |        |
                  |       |    |        |
              stream    |  (results  equipment
              state   WaterTAP  returned  geometry
                        sessions  to lead)
                      \   |    |    /
                       \  |    |   /
                       pe-deliverables
                            |
                      cost-estimator
```

Data flows between specialists via **stream state JSON files** and the
**artifact registry** — not by copying context windows. When pe-process
completes a sizing, it writes results to `mcp-outputs/streams/`. When
pe-deliverables generates an equipment list, it reads from stream state
files. The data is in the schema, not in the conversation.

---

## Boundary enforcement

The persona pattern only works if boundaries are enforced, not
suggested. PuranOS enforces them at three levels.

### Tool access (hard boundary)

Each persona's MCP configuration lists exactly which servers it can
access. pe-process cannot call procurement tools. cost-estimator cannot
modify stream state. This is not prompt-based enforcement — the tools
are simply not available to the wrong persona.

```
  pe-process.mcp.json (illustrative)
  ├── qsdsan-engine-mcp    (read/write)
  ├── water-chemistry-mcp   (read/write)
  └── openproject-mcp       (task coordination)

  cost-estimator.mcp.json (illustrative)
  ├── parametric-costing-mcp (read/write)
  ├── procurement-mcp        (read-only)
  └── openproject-mcp        (task coordination)
```

Stream state (formerly plant-state MCP) is now filesystem-based:
agents read and write JSON files at `mcp-outputs/streams/` using
the `plant-state-skill` conventions.

No amount of prompt injection can give pe-process access to
procurement-mcp. The server is not in its configuration.

### Side-effect governance (server-level)

When a persona sends an email, the MCP server enforces which identity
it sends from. When a persona creates a work package, the server
validates it has the required fields and appropriate project scope.
These constraints are in the tool implementation, not in the agent
instructions.

| Side effect         | Enforcement point          |
|---------------------|----------------------------|
| Email send          | Identity binding in office-mcp |
| Work package create | Field validation in openproject-mcp |
| Stream state write  | Schema validation via engineering-utils Pydantic models |
| File export         | Path scoping in filesystem server |

### Delegation protocol (behavioral)

When a persona encounters work outside its scope, it does not attempt
it. It creates a work package in OpenProject assigned to the appropriate
specialist. This keeps the audit trail clean and ensures each piece of
work is done by the persona with the right tool access and expertise.

The delegation protocol is behavioral, not structural. It depends on
the agent following its instructions. This is the weakest enforcement
layer — but it is backstopped by the hard tool-access boundary. Even
if a persona tries to do out-of-scope work, it lacks the tools.

---

## Beyond engineering

The same pattern applies across the firm. The scopes differ; the
structure is identical.

### Commercial

Sales, procurement, and cost estimation personas have CRM and
procurement tool access but no engineering tool access. A sales persona
can create an opportunity and delegate technical scoping to engineering,
but cannot modify design parameters.

### Operations

Maintenance, inventory, and compliance personas have CMMS, inventory,
and regulatory tool access. A maintenance persona can create work orders
and track assets but cannot modify project schedules or engineering
parameters.

### Back office

Bookkeeping, tax review, and legal personas have accounting and document
management access. Financial personas can generate reports but cannot
approve expenditures above policy thresholds.

### Cross-domain delegation

```
  Commercial ──(opportunity)──> Engineering ──(design)──> Operations
       |                             |                        |
       v                             v                        v
  CRM + procurement         stream state + engines     CMMS + compliance
```

Each domain boundary is a delegation point. A sales persona does not
call engineering tools — it creates a scoping task. An engineering
persona does not create purchase orders — it creates a procurement
request. The boundaries are structural, not advisory.

---

## Why not one agent per project

The alternative — a single agent with access to everything — is simpler
to implement. It fails at scale for three reasons.

**Context window limits.** The combined tool surfaces of QSDsan,
WaterTAP, procurement, CRM, CMMS, OpenProject, and accounting exceed
what a single agent can effectively use. Quality degrades as tool
surface area grows.

**Governance.** A single agent that can both size equipment and approve
purchase orders conflates responsibilities that must be separated for
auditability. Separation of duties is not optional in regulated
industries.

**Expertise framing.** A process engineering prompt and a procurement
prompt require different context, different heuristics, and different
output contracts. Personas make this explicit rather than hoping the
agent will switch modes correctly.

The research supports this selective decomposition. Multi-agent systems
outperform single agents only when each agent has a clear, bounded role
with explicit coordination. Adding agents without clear boundaries
produces *worse* results than a single agent. See
[Coordination and State Research](../../research/coordination-and-state.md).

---

## Relationship to other layers

Personas do not exist in isolation. They connect to every other
architectural layer in PuranOS.

| Layer               | Relationship to personas                        | Reference                                          |
|---------------------|-------------------------------------------------|----------------------------------------------------|
| Skills              | Scoped to personas. pe-deliverables runs the equipment-list skill, not pe-process. | [Skills as Expertise](../../approach/skills-as-expertise.md) |
| OpenProject         | Coordination substrate where personas delegate work to each other. | [Coordination Substrate](../../approach/coordination-substrate.md) |
| Engineering engines  | Accessed through personas. pe-process uses QSDsan; pe-separation uses WaterTAP. | [Engineering Engines](../../approach/engineering-engines.md) |
| Tool governance     | Enforced per persona at the MCP server boundary. | [Architecture Overview](../README.md) |
| Agent runtime       | Personas are instantiated as agent configurations with specific model, tools, and instructions. | [Agent Runtime](../agent-runtime/README.md) |

---

## Design principles

These principles govern how new personas are defined.

**Minimum viable scope.** A persona should have the smallest tool
surface that lets it complete its assigned work. Do not add tools
"in case they are needed."

**Single write domain.** At most one persona should have write access
to any given data domain at any given time. This prevents write
conflicts and makes state changes attributable.

**Explicit delegation.** If two personas need to collaborate, the
collaboration happens through OpenProject work packages, not through
shared context windows or direct calls.

**Composable, not hierarchical.** Personas can delegate to any other
persona, not just "down" a hierarchy. pe-separation can delegate back
to pe-process for water chemistry. The graph is a DAG per task, not a
fixed tree.

**Human-legible boundaries.** A non-technical project manager should be
able to read a persona's configuration and understand what it can and
cannot do. If the boundary requires reading source code to understand,
it is too complex.

---

## Extensibility

When the firm needs a new capability, a new persona is defined by the same four scopes: tool, expertise, authority, and boundary. The persona is ready when it can complete its assigned work without accessing tools outside its scope and without producing side effects outside its authority.

The cost of adding a persona is low — it is a configuration file, not a codebase. The cost of adding a persona *incorrectly* is high — an overly broad tool scope or missing boundary rule degrades the governance properties of the entire system. The design principles above exist to keep that cost visible.
