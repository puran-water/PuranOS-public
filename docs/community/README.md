# For Curious Industrial Engineers

## Why this exists

This documentation exists because the intersection of AI and industrial process engineering is underserved. Most AI platforms are built for software companies. Most engineering software is built without AI in mind. PuranOS is built at the intersection — and we publish the architecture because the reasoning behind it is worth sharing.

If you work in industrial wastewater, treatment design, or design-build project delivery, the problems described in these documents are probably familiar: fragmented tools, knowledge loss between projects, engineering models disconnected from project state, and the gap between what AI promises and what it actually delivers in an engineering context.

## Who this documentation is for

**Process engineers** who size treatment systems, select equipment, and produce engineering deliverables — and who wonder how AI could make that work more consistent and repeatable without replacing engineering judgment.

**Design-build project managers** who manage multi-discipline delivery across engineering, procurement, construction, and commissioning — and who see the coordination overhead that comes from disconnected systems.

**Engineering firm principals** who recognize that institutional knowledge walks out the door with senior engineers — and who are looking for a systematic way to capture and compound that expertise.

**Technical leaders** evaluating AI strategy for industrial firms — and who want to see an architecture that treats engineering computation and industrial standards as first-class concerns rather than afterthoughts.

## What you will find here

This is not a product pitch. There is no signup page, no demo request form, no pricing tier.

What you will find:

- **The architectural thesis** — why schema'd state beats memory systems for enterprise AI, backed by specific research findings. See [Schema Over Memory](../approach/schema-over-memory.md).

- **The coordination model** — why a PM tool (OpenProject) serves as the shared board for human+AI task delegation, and what the research says about agent coordination. See [Coordination Substrate](../approach/coordination-substrate.md).

- **The engineering model** — how deterministic simulation engines (QSDsan, WaterTAP) are treated as first-class infrastructure with session persistence, cross-engine handoffs, and model credibility metadata. See [Engineering Engines](../approach/engineering-engines.md).

- **The expertise capture model** — how reusable skills encode operating procedures so institutional knowledge compounds rather than evaporates. See [Skills as Expertise](../approach/skills-as-expertise.md).

- **The standards alignment** — how purpose-built schemas map to DEXPI, ISA-95, ISA 5.1, and other industrial consensus standards. See [Standards and Conformance](../approach/standards-and-conformance.md).

- **The research backing** — what specific studies say about agent coordination, memory systems, and multi-agent architectures, including the counter-evidence. See [Research Index](../research/README.md).

## What we value in collaborators

Domain expertise over software development. The hardest part of building an AI-native engineering platform is not the code — it is knowing what the right engineering workflow looks like, what parameters matter for a given design decision, and when a textbook answer is wrong.

If you have spent years sizing treatment systems, managing design-build projects, or developing engineering standards — you have knowledge that is difficult to replicate and valuable to encode.

We are not looking for code contributions. PuranOS is not open source. What we publish here is the architecture and the reasoning — so that people with relevant domain expertise can evaluate whether the approach is sound and whether a conversation makes sense.

## How to read these documents

Start with [Schema Over Memory](../approach/schema-over-memory.md) if you care about the AI architecture. It explains why PuranOS uses structured state schemas instead of retrieval-augmented memory, and what the tradeoffs are.

Start with [Engineering Engines](../approach/engineering-engines.md) if you care about the process engineering. It explains how simulation models are managed as persistent, auditable sessions rather than one-off calculations.

Start with [Coordination Substrate](../approach/coordination-substrate.md) if you care about project delivery. It explains how human engineers and AI agents share a single task board with clear ownership semantics.

Start with [Research Index](../research/README.md) if you want to see the evidence. Every architectural decision cites specific studies, and the counter-evidence is included.

## What this is not

This is not an open source project. There are no issues to file, no pull requests to submit, no contribution guidelines. The codebase is proprietary.

This is not a product demo. PuranOS is operational infrastructure for Puran Water's own design-build practice. It is not a SaaS product.

This is not a recruiting page. But if you are a process engineer who finds this interesting, that is worth knowing.

## Contact

If any of this resonates — whether you want to discuss the architecture, explore how it applies to your firm, or just compare notes on the intersection of AI and industrial engineering:

- **Puran Water LLC**
- **Hersh Kshetry**, Founder and Principal Engineer
- Website: [puranwater.com](https://puranwater.com/)
- Contact page: [puranwater.com/contact](https://puranwater.com/contact/)
- Schedule a call: [calendar.app.google/M1jzSdCB51sYiWux6](https://calendar.app.google/M1jzSdCB51sYiWux6)

No form to fill out. No sales funnel. Just a conversation between practitioners.
