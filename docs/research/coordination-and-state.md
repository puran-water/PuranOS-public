# What the Research Says About Agent Coordination

## Why this exists

PuranOS uses a specific coordination architecture: OpenProject as a shared board,
personas as bounded execution roles, and a hybrid state model separating
long-running collaborative state from execution reliability state.
These are not arbitrary choices -- they are responses to specific research findings.

This document presents the evidence.


---


## The llmenron exchange

The llmenron analysis (strangeloopcanon/llmenron) ran four progressive waves of
experiments testing how agents coordinate, store state, and delegate work.
The experiments used realistic scenarios with memory-dependent tasks, multi-turn
conversations, and both single-agent and multi-agent configurations.


### Wave 1-2: Explicit state beats scratchpad

The first experiments compared scratchpad-based memory (where agents maintain
free-form notes) against explicit thread_state (a structured object tracking
the current context of a conversation).

Results:

| Metric | Scratchpad | Explicit thread_state | Improvement |
|---|---|---|---|
| Memory-dependent target recovery | 0.213 | 1.000 | 4.7x |
| Target attachment accuracy | 0.835 | 0.976 | +17% |
| Token usage | baseline | ~1/3 of baseline | ~67% reduction |

The mechanism: scratchpad notes accumulate and the agent must search through
them. Explicit state puts the current context in a known structure. The agent
reads the state instead of searching memory.

This is the same mechanism that makes database queries superior to full-text
search for structured data. When the structure of the information is known,
encoding that structure is always better than leaving it as free text.

**PuranOS application:** Every coordination interaction uses structured state --
OpenProject work package fields for task-level state, PostgreSQL execution
ledger for reliability state, application schemas for domain facts.
Nothing is stored as free-form scratchpad.


### Wave 3: Shared board improves quality

The third wave tested shared boards -- structured objects where multiple agents
can read and write task state, visible to all participants.

Key findings:

- Shared board improved quality metrics for both single-agent and multi-agent
  configurations.
- Multi-agent without a shared board performed slightly worse than single-agent
  without one.
- The quality improvement from adding a shared board was larger than the quality
  improvement from adding more agents.

The stated conclusion: "do not build a swarm before you build a board."

This is a significant finding for enterprise AI architecture. It means the
coordination substrate matters more than the number of agents. Investing in a
good board (OpenProject, in PuranOS's case) yields more value than investing in
more agents or more sophisticated agent-to-agent communication.

**PuranOS application:** OpenProject IS the shared board. All persona-to-persona
delegation flows through it. There is no separate agent-to-agent communication
channel. See [Coordination Substrate](../approach/coordination-substrate.md).


### Wave 4: Actor identity eliminates unauthorized actions

The fourth wave added explicit role assignments on the board: who owns execution
(the internal worker) and who may respond externally (the public-facing
responder).

Results:

| Metric | Without actor identity | With actor identity |
|---|---|---|
| Unauthorized external responses | 33.3% | 0.0% |

When agents do not have explicit identity on the board, they sometimes act
outside their scope -- responding to external parties when they should only be
working internally, or making decisions that should require a different role's
authority.

**PuranOS application:** OpenProject's assignee role maps to "who executes."
The accountable role maps to "who approves/responds externally." Persona
configurations define what each role may do. This is not prompt-based
enforcement -- it is structural.


---


## Supporting literature

### Generative Agents (Park et al., 2023)

Memory streams with observation, reflection, and planning produce coherent
long-horizon agent behavior in simulated environments. Ablation study shows each
component contributes critically -- removing any one degrades performance
significantly.

**Relevance:** PuranOS achieves similar coherence through structured state rather
than memory streams. The PM tool provides observation (what tasks exist and their
status), the persona provides reflection (domain-specific judgment), and skills
provide planning (procedural workflows). The same functional decomposition,
different implementation.


### StateFlow (Wu et al., 2024)

Explicit state machines for LLM agent workflows improve success rates by 13-28%
at 3-5x lower cost compared to ReAct-style reasoning chains.

| Approach | Success Rate | Token Cost |
|---|---|---|
| ReAct | Baseline | Baseline |
| StateFlow (explicit states) | +13-28% | 20-33% of baseline |

**Relevance:** OpenProject's status workflows are explicit state machines. Work
packages move through defined states (draft, active, review, done) rather than
having agents reason about what state they are in.


### Agent Workflow Memory (Wang et al., 2024)

Reusable workflows extracted from successful task completions give 24.6-51.1%
relative improvement on Mind2Web and WebArena benchmarks.

The key finding: procedural memory (reusable workflows) outperforms episodic
memory (replaying past interactions). An agent that knows "how to do X" performs
better than an agent that remembers "what happened last time I did X."

**Relevance:** PuranOS skills ARE reusable workflows. They encode the procedural
knowledge extracted from successful engineering work.
See [Skills as Expertise](../approach/skills-as-expertise.md).


### TapeAgents (Saveliev et al., 2024)

Structured session logs as first-class artifacts: resumable state, debugging
records, and training data. Sessions are not just logs -- they are replayable
execution traces.

**Relevance:** PuranOS's PostgreSQL execution ledger records state transitions,
tool calls, and side effects as structured data. These records serve the same
function: debugging, audit, and potential future training.


### LLM-Based Multi-Agent Blackboard System (2025)

A shared blackboard (structured common workspace) shows 13-57% relative
improvement over RAG-based and master-slave baselines in collaborative
problem-solving. Note: this paper evaluates **data-science information discovery
over data lakes**, not industrial project delivery. The general principle — that
shared workspaces improve coordination — transfers, but the specific domain and
performance numbers do not directly validate an OpenProject-centered coordination
layer for industrial engineering.

**Relevance:** Supports the shared board pattern as a coordination primitive.
OpenProject serves as PuranOS's blackboard — a structured workspace where all
personas can read context and write results. The extrapolation from data-science
tasks to industrial project coordination remains an architectural inference, not
a directly demonstrated result.


### Voyager (Wang et al., 2023)

An ever-growing skill library with iterative self-verification produces large
performance gains over stateless agent operation. Skills are code-based,
versioned, and composable.

**Relevance:** PuranOS's skill library follows the same pattern -- a growing
collection of procedural knowledge that compounds over time.
See [Skills as Expertise](../approach/skills-as-expertise.md).


### AWS Multi-Agent Collaboration

Production multi-agent system achieving up to 90% goal success with 23% lift
from payload referencing (passing structured data between agents rather than
free-text summaries) and latency reduction from dynamic routing.

**Relevance:** Validates that structured payloads between agents (which PuranOS
achieves via schema'd state in OpenProject and plant-state) outperform free-text
handoffs.


---


## Counter-evidence

The research is not uniformly positive. PuranOS takes the following
counter-evidence seriously.


### "Why Do Multi-Agent LLM Systems Fail?" (2025)

Studies multiple multi-agent frameworks and reports high failure rates (41% to
86.7%) across evaluated SOTA open-source multi-agent systems. These are
benchmarked MAS traces, not a universal statement about all production
multi-agent systems. Dominant failure modes:

- **Cascading errors.** One agent's mistake propagates through the system.
- **Ambiguous task boundaries.** Agents duplicate work or leave gaps.
- **Hallucination propagation.** Fabricated outputs from one agent are treated
  as facts by downstream agents.
- **Memory/state management.** Inconsistent context across agents.

Memory and state management fixes help but are inconsistent -- they improve some
failure modes while creating others.

**PuranOS response:** Tight persona boundaries prevent cascading errors
(pe-mechanical cannot overwrite pe-process's results). Explicit delegation via
OpenProject prevents ambiguous task boundaries. Schema validation prevents
hallucination propagation (a fabricated equipment spec fails schema validation).
Structured state prevents memory management failures.


### "Rethinking the Value of Multi-Agent Workflow" (2026)

Shows that a single agent with the right tools can match multi-agent performance
at lower cost, thanks to KV-cache reuse and preserved context.

**PuranOS response:** PuranOS uses multiple agents only where domain
heterogeneity, tool surface area, or governance constraints actually require
them. The PE team has specialists not for architectural elegance but because the
combined tool surface of QSDsan, WaterTAP, CFD, and cost estimation exceeds what
one agent can effectively use. For simpler workflows, a single persona handles
the work.


### "More Agents Is All You Need" (2024)

Some multi-agent performance gains are ordinary ensemble/sampling effects --
multiple independent attempts at the same task with best-answer selection. This
does not require agent specialization or coordination, just parallel execution.

**PuranOS response:** PuranOS personas are not doing the same task in parallel.
They are doing different tasks with different tools. The value comes from
specialization and boundary discipline, not from ensemble effects.


### MAP: Measuring Agents in Production (2025)

Survey of production agent deployments (with varying sample bases per finding) reports:

| Finding | Percentage | Sample base |
|---|---|---|
| Use manual prompt construction (not frameworks) | 79% | Survey respondents |
| Limit agents to 10 or fewer steps | 68% | Survey respondents |
| Use no agent frameworks (LangChain, CrewAI, etc.) | 85% | 17 of 20 production case studies |
| Rely on human-in-the-loop evaluation | 74% | Survey respondents |

Note: the 85% "custom over frameworks" figure comes from 17 of 20 production
case studies, not the same survey base as the other percentages.

**PuranOS response:** This validates several PuranOS design choices. Skills are
"manual prompt construction" -- they define exactly what tools to call and in
what order. Workflows are shallow (typically 5-10 steps). No agent frameworks
are used. Human review gates are explicit (the accountable role in OpenProject).


### AgentArch (2025)

The best model/configuration combination achieves only 70.8% on the simpler
time-off task and 35.3% on the harder customer-routing task. Multi-agent ReAct
configurations perform especially poorly. However, the paper shows
model-specific preferences — GPT-4.1-mini had single-agent ReAct settings that
beat some function-calling settings — so the relationship between architecture
and performance is not universal.

**PuranOS response:** PuranOS uses function-calling, not ReAct. Agents call
typed MCP tools directly rather than reasoning through chains of thought about
which tool to use. This is consistent with AgentArch's general finding that
ReAct-style configurations tend to underperform, while acknowledging that the
optimal architecture may vary by model and task complexity.


---


## Synthesis

The research converges on four principles that PuranOS implements.

### 1. Explicit state over implicit memory

Structured state objects outperform scratchpad, free-form memory, and RAG at
every scale tested. The improvement is not marginal -- it is often 3-5x on
memory-dependent tasks.

PuranOS stores all coordination state in typed fields: OpenProject work package
attributes, PostgreSQL ledger rows, JSON-schema'd domain objects. No agent
maintains a free-form memory of what has happened.


### 2. Shared board over message passing

A structured common workspace improves quality for all configurations.
Adding more agents without a board makes things worse.

OpenProject is the board. Every task assignment, status transition, and handoff
is a board operation. There is no message bus, no pub/sub, no agent-to-agent
RPC. Agents read the board, do their work, and write results back to the board.


### 3. Procedural memory over episodic memory

Reusable workflows outperform replay of past interactions for cross-task
generalization. An agent with a skill library outperforms an agent with a
conversation history.

PuranOS skills encode procedural knowledge as executable instructions. They are
versioned, composable, and domain-specific. They are not recordings of past
sessions -- they are distilled expertise.
See [Skills as Expertise](../approach/skills-as-expertise.md).


### 4. Bounded agents over universal agents

Multiple specialized agents outperform a single universal agent only when each
agent has clear scope, explicit identity, and structured coordination.
Without these, multi-agent systems underperform single-agent baselines.

PuranOS personas have explicit tool surfaces, defined domains, and structural
boundaries enforced by OpenProject roles. The PE team has specialists because
specialization is required by the problem, not because more agents is better.
See [Persona Boundaries](../architecture/personas/README.md).


---


## The cost of getting it wrong

The counter-evidence section is not a formality. The failure modes described
in the literature are common in production agent systems.

| Failure mode | Frequency in literature | PuranOS mitigation |
|---|---|---|
| Cascading errors | 41-87% failure rate in evaluated SOTA MAS | Persona boundaries, schema validation |
| Ambiguous task boundaries | Dominant failure mode | Explicit delegation via OpenProject |
| Hallucination propagation | Observed across all systems | Schema'd state, typed tool outputs |
| Memory inconsistency | Observed across all systems | Single source of truth (OpenProject + PostgreSQL) |
| Unauthorized actions | 33% without identity | Actor identity on board |
| Framework overhead | 85% of production systems avoid | No frameworks, direct MCP tool calls |

The pattern is consistent: in the studied settings, systems that treat coordination
as a prompt engineering problem perform poorly. Systems that treat coordination as a
state management problem perform substantially better.

PuranOS treats coordination as a state management problem.


---


## Further reading

- [Schema Over Memory Research](schema-over-memory.md) -- Research on why
  schema'd state beats memory systems
- [Coordination Substrate](../approach/coordination-substrate.md) -- How PuranOS
  applies these findings
- [Persona Boundaries](../architecture/personas/README.md) -- How bounded agents
  work in practice
- [Skills as Expertise](../approach/skills-as-expertise.md) -- Procedural memory
  in practice
