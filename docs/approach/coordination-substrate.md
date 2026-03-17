# OpenProject as Coordination Substrate

## Why this exists

Multi-agent systems need shared state. But shared state implemented as
scratchpads or prompt memory degrades quality. The research is clear on
this point: unstructured shared context produces worse results than no
sharing at all.

The question is not whether agents need coordination — it is what
substrate that coordination should use. PuranOS's answer: a project
management tool, specifically OpenProject, serving as the shared board
for human+AI bidirectional task delegation.

This document explains the research basis for that choice, what
OpenProject provides that custom alternatives would not, and where the
design deliberately stops.

---

## The research finding

The llmenron analysis (strangeloopcanon/llmenron) stress-tested agent
architectures across four iterative waves of experiments. The findings
shaped PuranOS's coordination model directly.

**Wave 1-2: Explicit state beats scratchpad.**
Moving from scratchpad-only memory to explicit `thread_state` — a
structured object tracking the current context of a conversation —
produced dramatic improvements:

- Memory-dependent target recovery: 0.213 to 1.000
- Target attachment accuracy: 0.835 to 0.976
- Token usage: approximately 2/3 reduction

The mechanism is straightforward. A scratchpad accumulates free-form
notes that the agent must search through. Explicit state puts the
current context in a known structure. The agent does not need to
"remember" what is happening — it reads the state.

**Wave 3: Shared board improves quality for both single and multi-agent.**
Adding a shared board (a structured place where multiple agents can read
and write task state) improved quality metrics for both configurations.
The critical finding: multi-agent without a board performed *slightly
worse* than single-agent without one.

The stated conclusion: "do not build a swarm before you build a board."

**Wave 4: Actor identity eliminates unauthorized actions.**
Assigning explicit roles on the board — who owns execution, who may
speak externally, who must approve — reduced unauthorized responses from
33.3% to 0.0%.

This maps directly to the PM concept of "assignee" (who does the work)
and "accountable" (who approves the result).

---

## Why OpenProject

PM tools already model the coordination primitives that industrial firms
need:

| PM Concept | What It Represents | Why It Matters for Agent Coordination |
|---|---|---|
| Work package | A discrete unit of work | Stable task identity across sessions |
| Assignee + Accountable | Who does vs. who approves | Actor identity (owner vs. responder) |
| Status workflow | Draft, active, review, done | Explicit state machine |
| Predecessor/successor | Dependency between tasks | Blocking semantics |
| Hierarchy (parent/child) | Work decomposition | Task scoping |
| Comments/journal | Running discussion | Audit trail |
| Custom fields | Typed metadata | Agent-specific state (persona, context, scope) |
| Project membership | Who can see what | Boundary enforcement |

OpenProject was chosen specifically because it provides:

- **Distinct assignee and accountable roles** — mapping to the
  research's actor-identity finding.
- **Rich predecessor/successor semantics** — industrial projects are
  inherently dependency-heavy.
- **Open-source, self-hosted, Postgres-backed** — consistent with the
  schema-over-memory thesis.
- **Both humans and agents see the same board** — no separate "agent
  dispatch" system.

The alternative — building a custom task dispatch system — would
replicate what PM tools already do, without the human-facing UI,
reporting, and audit trail.

---

## The minimum viable "job board"

Per the research, effective agent coordination requires each work item
to carry a specific set of fields. The table below maps each requirement
to OpenProject's native coverage:

| Field | Purpose | OpenProject Coverage |
|---|---|---|
| Stable task ID | Reference across sessions and systems | Native (work package ID) |
| Source object link | What triggered this work | Custom field or relation |
| Task state | Current lifecycle position | Status workflow |
| Priority / SLA | Urgency and deadlines | Priority field + due date |
| Internal owner | Who executes | Assignee role |
| External responder/signer | Who approves or responds externally | Accountable role |
| Permitted action scope | What the executor may do | Custom field (persona-defined) |
| Canonical context payload | The information needed to execute | Description + custom fields |
| Dependencies/blockers | What must complete first | Predecessor relations |
| Audit trail | What happened and when | Journal entries (automatic) |
| Escalation/approval | When human review is required | Status transitions + accountable role |

OpenProject covers most of this natively. The gaps (permitted action
scope, canonical context payload) are filled with typed custom fields.
No bespoke dispatch layer is needed.

---

## The hybrid state model

PuranOS deliberately separates two state substrates. Each handles a
different class of concern:

```
┌─────────────────────────────────────────────────────────┐
│  OpenProject: Long-running collaborative state          │
│  - Task hierarchy and decomposition                     │
│  - Predecessor/successor dependencies                   │
│  - Comments, approvals, status transitions              │
│  - Human-readable project history                       │
│  - Typed custom fields for agent context                │
│                                                         │
│                         ↕                               │
│                                                         │
│  PostgreSQL: Execution reliability state                │
│  - Ingress event deduplication                          │
│  - Work item attempt tracking (retries, leases)         │
│  - State transition audit log                           │
│  - Side-effect records (emails sent, APIs called)       │
│  - Declarative schedule execution                       │
└─────────────────────────────────────────────────────────┘
```

This split exists because the two substrates have different
requirements:

| Requirement | OpenProject | PostgreSQL Execution Ledger |
|---|---|---|
| Human legibility | Primary concern | Not required |
| Exact-once semantics | Not designed for this | Designed for this |
| Long-term history | Months to years | Hours to days |
| Schema flexibility | Custom fields, types | Rigid execution schema |
| Who reads it | Humans + agents | Agents + audit |

Combining these into one system would compromise both. OpenProject would
become cluttered with execution minutiae. The execution ledger would
lose its exact semantics to accommodate human-facing features.

See [Schema Over Memory](schema-over-memory.md) for the reasoning behind
schema'd state as the foundation layer.

---

## Delegation directions

All delegation flows through the same OpenProject mechanism. There is no
separate dispatch system, no message queue between agents, no shared
prompt buffer. The PM tool is the dispatch system.

**Human to Agent.** A project manager assigns a work package to a
persona (e.g., pe-process). The communication agent picks it up, loads
the persona's config, and executes.

**Agent to Agent.** pe-lead creates child work packages and assigns them
to specialist personas (pe-separation, pe-mechanical, cost-estimator).
Each specialist picks up their work independently. The parent work
package tracks overall progress.

**Agent to Human.** An agent sets a work package status to "review" and
assigns the accountable role to a human engineer. The human reviews,
comments, and either approves or requests changes through the same
interface they use for all other project work.

The key property: none of these flows require special infrastructure.
They all use the same work package lifecycle, the same role assignments,
the same status transitions. A human reviewing agent-created work uses
the same UI they use for human-created work.

---

## How actor identity maps to personas

The research's Wave 4 finding — that explicit actor identity eliminates
unauthorized actions — maps directly to PuranOS's persona model:

```
┌──────────────────┐     ┌──────────────────┐
│  OpenProject     │     │  PuranOS Persona  │
│  Role            │     │  Config           │
├──────────────────┤     ├──────────────────┤
│  Assignee        │ ──→ │  Executor         │
│  (does the work) │     │  (which agent +   │
│                  │     │   which tools)    │
├──────────────────┤     ├──────────────────┤
│  Accountable     │ ──→ │  Approver         │
│  (signs off)     │     │  (human or lead)  │
└──────────────────┘     └──────────────────┘
```

Each persona config specifies:

- Which MCP servers it may call (tool surface).
- Which OpenProject projects it may read and write.
- What status transitions it may perform.
- Whether it may create child work packages (delegation authority).

These constraints are enforced at config load time, not at runtime
through prompt engineering. The agent cannot call tools it was not given.
It cannot assign work packages to projects it cannot see.

---

## Supporting literature

Several research threads converge on the same finding — that explicit,
structured coordination substrates outperform implicit coordination:

**Generative Agents** (Park et al.). Memory streams with reflection
produce coherent long-horizon agent behavior. Ablation shows each
component (observation, reflection, planning) contributes critically.
PuranOS gets the same effect from PM tool state rather than in-memory
reflection chains.

**StateFlow** (Wu et al.). Explicit state machines improve success rates
by 13-28% at 3-5x lower cost vs. ReAct-style reasoning. OpenProject's
status workflows are explicit state machines with the same properties.

**Agent Workflow Memory** (Wang et al.). Reusable workflows extracted
from successful runs give 24.6-51.1% relative improvement. PuranOS
captures these as skills — codified procedures that agents execute
against the board.

**TapeAgents** (Saveliev et al.). Structured session logs as resumable
state, providing debugging and training artifacts. PuranOS's execution
ledger serves the same function, with the addition that the
human-readable layer lives in OpenProject.

**LLM-Based Multi-Agent Blackboard System**. Shared blackboard shows
13-57% improvement over RAG and master-slave baselines. OpenProject is
the blackboard.

---

## Counter-evidence and how PuranOS responds

The research also contains significant warnings. Ignoring them would be
negligent.

**"Why Do Multi-Agent LLM Systems Fail?"** reports high failure rates
across multiple studied multi-agent frameworks. The dominant failure modes: cascading
errors between agents, ambiguous task boundaries, and unchecked
hallucination propagation. Memory and state management fixes help but
are inconsistent.

**"Rethinking the Value of Multi-Agent Workflow"** shows that a single
agent with the right tools can match multi-agent performance at lower
cost, thanks to KV-cache reuse and preserved context.

**MAP (Measuring Agents in Production)** finds (across varying sample bases) that most production
agent deployments use manual prompt construction, limit agents to 10 or
fewer steps, use no agent frameworks, and rely on
human-in-the-loop evaluation.

PuranOS responds to this evidence with four specific design choices:

1. **Multiple agents only where domain heterogeneity requires it.** The
   PE team has specialists not for architectural elegance but because a
   single agent cannot hold QSDsan, WaterTAP, CFD, and cost estimation
   tool surfaces simultaneously while maintaining quality. Where one
   agent suffices, one agent is used.

2. **Function-calling over ReAct.** Agents call typed MCP tools directly
   rather than reasoning through multi-step chains of thought about
   which tool to use. This reduces the token cost and error surface
   that ReAct-style architectures introduce.

3. **Shallow workflows with human gates.** Most agent workflows are 5-10
   steps with explicit approval points, not deep autonomous chains. This
   directly addresses the cascading-error finding.

4. **The shared board as coordination primitive.** Rather than agents
   communicating through prompts or message passing, they coordinate
   through typed state on the OpenProject board — exactly the pattern
   the research supports.

---

## What this is not

This document does not claim that OpenProject is the only viable
substrate. Any system providing stable task identity, typed state,
role-based access, and dependency semantics could serve the same
function.

It also does not claim that the coordination substrate replaces all other
state. The execution ledger (PostgreSQL) handles reliability concerns
that no PM tool is designed for. MCP server state handles simulation
sessions. The coordination substrate handles *coordination* — who does
what, in what order, with whose approval.

The distinction matters. Overloading any one substrate with concerns it
was not designed for degrades all of them.

---

## Further reading

- [Schema Over Memory](schema-over-memory.md) — why schema'd state is the foundation
- [Coordination and State Research](../research/coordination-and-state.md) — detailed research analysis
- [Architecture Overview](../architecture/README.md) — how the hybrid state model is implemented
