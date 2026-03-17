# Why Skills Compound

## Why this exists

Senior engineers leave. Project knowledge evaporates. Every new project reinvents sizing procedures, equipment selection logic, and deliverable formats. The gap is not "lack of AI" — it is lack of captured, repeatable expertise.

Industrial engineering firms are expertise businesses. The difference between a good design and a bad one is not access to software — it is knowing which parameters matter, what heuristics apply, and when to deviate from textbook answers. That knowledge lives in senior engineers' heads, accumulated over decades of projects.

When those engineers leave, the knowledge leaves with them. New engineers learn by doing, repeating mistakes that were already solved. Each project rediscovers sizing approaches, equipment selection criteria, and deliverable formats that previous projects already figured out.

Skills solve this by making institutional knowledge machine-executable.


## What a skill is

A skill is a markdown file that encodes an operating procedure. It specifies:

- Which MCP tools to call
- In what order
- With what parameters and validation checks
- What the output contract looks like
- What risk boundaries apply

A skill is not a prompt template. Prompt templates describe what you want. Skills describe how to get it reliably. The difference matters because repeatability and auditability are non-negotiable in engineering.

A skill is not a script. Scripts are brittle — they break when inputs vary. Skills are declarative procedures that an agent executes within persona boundaries, adapting to the specific project context while following the defined workflow.


### Anatomy of a skill

```
┌─────────────────────────────────────────────┐
│  Frontmatter                                │
│  - name, description, trigger phrases       │
│  - target persona                           │
│  - tool dependencies (MCP servers used)     │
├─────────────────────────────────────────────┤
│  MCP Tool Map                               │
│  - which tools are called, from which       │
│    servers, for what purpose                │
├─────────────────────────────────────────────┤
│  Execution Workflow                         │
│  - ordered steps with tool calls            │
│  - validation checks between steps          │
│  - conditional branches                     │
├─────────────────────────────────────────────┤
│  Output Contract                            │
│  - required sections in the output          │
│  - data provenance requirements             │
│  - format specifications                    │
├─────────────────────────────────────────────┤
│  Risk Boundaries                            │
│  - what the skill must NOT do               │
│  - when to escalate to a human              │
│  - credibility constraints on inputs        │
└─────────────────────────────────────────────┘
```


### Illustrative example: Equipment list generation

An equipment list skill encodes the workflow a senior engineer follows:

1. Query the plant-state registry for the current design.
2. Extract equipment from each process unit's sizing results.
3. Validate each item against the equipment-item schema (tag format, required attributes, capacity units).
4. Assign ISA 5.1-compliant tags using the area-code-sequence pattern (e.g., `340-B-01A`).
5. Aggregate across all process areas.
6. Generate the deliverable in the required format with a typed artifact envelope.
7. Register the artifact with dependency tracking.

The skill does not contain the engineering knowledge to size the equipment — that belongs to the engineering engines. It contains the procedural knowledge to assemble, validate, tag, format, and register the results. That procedural knowledge is what disappears when a senior engineer leaves.


## How skills differ from prompts

| Aspect | Prompt | Skill |
|---|---|---|
| Specifies | What you want | How to get it |
| Reproducibility | Variable — depends on phrasing | Consistent — same steps every time |
| Auditability | Difficult — output varies | Traceable — each step maps to a tool call |
| Composability | Limited — prompts are self-contained | High — skills reference other skills and shared schemas |
| Knowledge capture | None — the requester must know what to ask | Complete — the workflow IS the captured knowledge |
| Evolution | Rewrites on each use | Versioned and improved over time |


## Why skills compound

Each successful project produces workflows that can be encoded as skills.

The first MLE bioreactor sizing uses a senior engineer's judgment at every step. The skill captures that judgment. The second MLE sizing follows the skill. It takes less time and produces a more consistent result. After five projects, the skill has been refined with edge cases, better heuristics, and improved output formats. After twenty projects, the skill represents the accumulated institutional knowledge of the firm — not just one engineer's approach, but the refined consensus of many projects.

This compounding effect is the core value proposition. The firm gets better at its work over time, regardless of staff turnover, because the expertise is encoded in skills rather than trapped in individuals.

```
Project 1:  Senior engineer → manual workflow → result
Project 2:  Skill v1 → faster, more consistent result
Project 5:  Skill v3 → refined with edge cases
Project 20: Skill v8 → institutional knowledge, not individual expertise
```

The compounding is not just per-skill. Skills compose. A "preliminary design report" skill calls the "equipment list" skill, the "mass balance summary" skill, and the "regulatory compliance check" skill. Improvements to any component skill propagate to every composite skill that references it. The library grows in breadth and depth simultaneously.


## How skills are captured

Skills are markdown files. Authoring a skill requires domain expertise, not software development ability.

An experienced process engineer who can describe "how I size an MLE bioreactor" — which parameters to check, what heuristics to apply, when to use conservative vs. aggressive assumptions, what the output should include — produces a more valuable skill than a software developer who can write Python.

This is intentional. The bottleneck in industrial engineering is domain expertise, not code. The skill format is designed so that domain experts author them directly.

A typical capture path:

1. An engineer completes a task manually, working with an agent.
2. The successful workflow is extracted into a candidate skill.
3. A second engineer reviews the skill for correctness and completeness.
4. The skill is added to the library and available to all agents and personas.

The review step matters. Skills encode best practices. A skill that encodes a bad practice propagates that bad practice across every future project. Peer review catches errors before they compound.

For details on skill structure, see [The Skill Model](../community/skill-contribution-guide.md).


## Skill versioning and trust

Skills evolve. A v1 skill is a first draft — it captures the basic workflow but may miss edge cases. A v8 skill has been tested against dozens of project contexts and refined at each step.

Version history matters for auditability. When a deliverable is produced using a skill, the skill version is recorded. If a design decision is questioned later, the exact procedure that produced it can be reconstructed.

Trust in a skill grows with its version history. A newly contributed skill is useful but unproven. A skill that has been through twenty projects and eight revisions has earned a level of institutional trust that no individual engineer's memory can match — it has been tested, corrected, and refined by multiple practitioners across varied project conditions.

This is related to the credibility model described in [Schema Over Memory](schema-over-memory.md). Just as simulation results carry credibility metadata, skills carry implicit credibility through their version and usage history.


## Research alignment

The skills-as-expertise pattern aligns with several research findings.

**Voyager** (Wang et al.) demonstrated that an ever-growing skill library enables agents to solve progressively harder tasks by composing previously learned capabilities. The key insight: procedural memory (skills) compounds in ways that episodic memory (logs of past events) does not.

**Agent Workflow Memory** (Wang et al.) showed that reusable workflows extracted from successful task completions give 24.6-51.1% relative improvement on web navigation benchmarks. Workflows beat episodic replay because they generalize across similar tasks.

**TapeAgents** (Saveliev et al.) formalized the idea of structured session artifacts — logs that are not just records but replayable, debuggable, and trainable. PuranOS skills serve a similar function: they are the formalized, reusable version of successful execution patterns.

The literature consistently shows that procedural memory beats episodic memory for task performance. Skills are procedural memory for the firm.


## What skills are not

Skills are not automation scripts. They do not replace engineering judgment — they encode the procedural framework within which judgment is applied. An agent executing a skill still makes decisions at each step. The skill constrains and guides those decisions; it does not eliminate them.

Skills are not chatbot prompts. They do not describe what to say — they describe what to do. The distinction is operational. A prompt produces text. A skill produces validated engineering artifacts.

Skills are not rigid. An agent executing a skill adapts to the specific project context. The skill defines the workflow; the agent applies it to the current situation. If a validation check fails, the agent does not blindly continue — it escalates or adjusts within the boundaries the skill defines.

Skills are not a substitute for engineering engines. The engines perform simulation and computation. Skills orchestrate those engines into coherent workflows. See [Engineering Engines](engineering-engines.md) for the computation layer.


## The organizational shift

Traditional firms store expertise in three places: senior engineers' heads, project files from past work, and informal mentorship. All three are fragile. People leave. Project files are unstructured and hard to search. Mentorship depends on proximity and willingness.

Skills add a fourth storage mechanism: machine-executable procedures that persist independent of any individual. This does not eliminate the need for experienced engineers — someone must author and review the skills. It changes the economics of expertise retention. A departing senior engineer who has contributed twenty skills leaves behind twenty encoded workflows that continue to benefit the firm.

The shift is from expertise as a scarce, perishable resource to expertise as a durable, compounding asset.


## Further reading

- [Engineering Engines](engineering-engines.md) — the computation layer that skills orchestrate
- [Schema Over Memory](schema-over-memory.md) — why typed schemas beat unstructured context
- [Persona Boundaries](../architecture/personas/README.md) — how skills are scoped to personas
- [The Skill Model](../community/skill-contribution-guide.md) — skill anatomy and structure
