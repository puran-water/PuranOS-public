# Knowledge Wiki for Unstructured Context

## Why this exists

The schema-over-memory thesis holds for canonical objects — equipment, streams,
tasks, costs, vendor records. These are structured entities with typed fields,
known relationships, and clear ownership. A properly schema'd database is the
right home for them.

But not all relevant context fits in a schema'd database.

Meeting synthesis. Research papers. Competitive intelligence. Design rationale.
Commissioning lessons learned. Vendor evaluation narratives. Regulatory
interpretation notes. Post-mortem analyses. These are knowledge artifacts —
unstructured by nature, valuable in aggregate, and poorly served by both
traditional databases and naive RAG systems.

They do not belong in OpenProject (which holds task state, not reference
knowledge). They do not belong in the CRM (which holds relationship state, not
technical context). They do not belong in Paperless-NGX (which holds documents
needing a document ID for cross-system reference, not synthesized knowledge).
And they do not belong in a vector store (which fragments them into chunks and
loses the relationships between ideas).

The Knowledge Wiki closes this gap. It is an Obsidian-based system where LLM
agents ingest raw sources, compile them into linked markdown articles with
backlinks, and iteratively enhance the wiki through compilation, Q&A, and
linting. The result is a navigable, interlinked knowledge base that agents can
query and humans can browse — without the maintenance burden of a custom
knowledge graph or the retrieval noise of a vector store.

The inspiration is Karpathy's LLM Knowledge Base pattern: use LLMs not just to
retrieve information but to compile and maintain a structured knowledge
repository from raw sources, producing artifacts that are more useful than the
originals.


---


## The pattern: raw sources to linked knowledge

The wiki operates through a defined pipeline, not ad-hoc note-taking.

```
Raw sources (transcripts, papers, reports, emails)
            ↓
LLM compilation (extract, synthesize, link)
            ↓
Linked wiki articles (markdown + backlinks + metadata)
            ↓
Q&A layer (agents query the wiki for context)
            ↓
Linting (scheduled consistency checks, dead link removal, stale content flags)
```

**Ingestion.** Raw sources arrive from multiple channels. Meeting transcripts
come from the communication agent. Research papers are dropped into inbox
folders. Vendor evaluation notes are produced during procurement workflows.
Design rationale is captured during engineering reviews.

**Compilation.** LLM agents process raw sources into structured wiki articles.
This is not summarization — it is synthesis. A meeting transcript about a
client's influent characteristics produces not a summary of the meeting but
updates to the project's water quality article, the client relationship article,
and potentially new articles on contaminants or treatment challenges that were
discussed.

**Linking.** Every article is connected to related articles through Obsidian
backlinks. A treatment technology article links to the projects where it was
applied, the vendors who supply it, and the regulatory context that constrains
it. These links are maintained by the compilation agents, not by humans.

**Q&A.** Agents query the wiki when they need context that is not available in
schema'd databases. Before sizing a treatment system, the process engineering
agent can check the wiki for lessons learned from similar projects, known
vendor issues, or regulatory nuances specific to the jurisdiction.

**Linting.** Scheduled maintenance passes check for broken links, stale content
(articles not updated in configurable timeframes), orphaned articles (no
backlinks), and internal consistency issues. The linting agents flag issues;
human review determines the resolution.


---


## Relationship to Paperless-NGX

The boundary between Paperless-NGX and the Knowledge Wiki is deliberate and
load-bearing.

| Concern | Paperless-NGX | Knowledge Wiki |
|---------|---------------|----------------|
| What it holds | Documents needing a document ID | Synthesized knowledge articles |
| Examples | Permits, vendor submittals, contracts, P&IDs, certificates | Meeting synthesis, design rationale, competitive intel, lessons learned |
| Primary consumer | Cross-system reference (linked from schema'd records) | Agent context and human browsing |
| Identity model | Document ID, tags, correspondent, document type | Wiki article path, backlinks, metadata frontmatter |
| Retrieval pattern | By document ID or structured search | By topic, backlink traversal, or full-text query |
| Authorship | Humans upload; OCR extracts | Agents compile from raw sources; humans review |

A vendor submittal goes into Paperless because it needs a document ID that the
procurement schema can reference. The lessons learned from reviewing that
submittal — what was strong, what was missing, how it compared to competitors
— go into the wiki because they are knowledge to be synthesized and linked, not
a document to be filed.

The two systems complement each other. Paperless is the filing cabinet. The wiki
is the institutional memory.


---


## Multi-vault architecture

Not all knowledge should be co-located. Competitive intelligence should not be
browsable alongside engineering design notes. Client-specific project knowledge
should not persist in the operations vault after project closeout.

The wiki uses a multi-vault architecture separated by concern:

| Vault | Purpose | Lifecycle |
|-------|---------|-----------|
| Operations | Operational procedures, maintenance patterns, vendor relationships | Permanent, continuously updated |
| Engineering | Design rationale, technology evaluations, lessons learned, standards interpretation | Permanent, grows with project history |
| Commercial | Competitive intelligence, market research, client relationship context, sales playbooks | Permanent, access-restricted |
| Project (ephemeral) | Project-specific knowledge during active delivery | Provisioned at project start, archived at closeout |

Ephemeral project vaults are provisioned on demand when a new project begins.
They accumulate project-specific knowledge — client conversations, site-specific
constraints, design decisions and their rationale — without polluting the
permanent vaults. At project closeout, relevant knowledge is graduated into the
permanent vaults and the project vault is archived.

Vault isolation also enables access control. The commercial vault contains
competitive intelligence that should not be accessible to all personas. The
engineering vault contains technical depth that the sales persona does not need
in its context window. Separation by concern keeps each agent's context focused.


---


## Implementation: two paths, shared filesystem

The wiki is served through two independent interfaces that share a common
filesystem of markdown files.

```
┌────────────────────────────────────────────────────────┐
│  Markdown files on disk (the single source of truth)   │
│                                                        │
│     ┌──────────────────┐    ┌───────────────────┐     │
│     │  Obsidian Docker  │    │  mcpvault MCP     │     │
│     │  (human UI)       │    │  (agent API)      │     │
│     │                   │    │                   │     │
│     │  Browse, search,  │    │  15 tools for     │     │
│     │  graph view,      │    │  read, write,     │     │
│     │  visual editing   │    │  search, link,    │     │
│     │                   │    │  lint, query      │     │
│     └──────────────────┘    └───────────────────┘     │
│            ↑                         ↑                 │
│         Humans                    Agents               │
└────────────────────────────────────────────────────────┘
```

**Obsidian Docker** provides the human-facing UI. Engineers and managers browse
the wiki through Obsidian's web interface — graph view for exploring
connections, search for finding specific topics, and the familiar markdown
editor for the occasional human contribution. Obsidian is read-heavy from the
human side; agents are the primary authors.

**mcpvault** is the MCP server that gives agents filesystem-native access to
the wiki. It provides 15 tools covering article creation, content updates,
backlink management, full-text search, metadata queries, and linting
operations. The server operates directly on the markdown filesystem — no
intermediate database, no sync layer, no eventual consistency issues. What the
agent writes is immediately visible in Obsidian, and vice versa.

The filesystem-native approach is a deliberate choice. Markdown files are
inspectable, version-controllable, and portable. If the MCP server fails, the
wiki is still a directory of readable text files. If Obsidian is unavailable,
agents still have full access through mcpvault. Neither path depends on the
other.


---


## Scheduled maintenance

The wiki is not a write-once artifact. It requires ongoing maintenance to remain
useful, and that maintenance is automated.

**Nightly reconciliation.** A scheduled agent pass checks for new raw sources
that have not been compiled, articles that reference outdated information, and
backlinks that point to moved or deleted articles. Repairs are made
automatically where confidence is high; flagged for human review otherwise.

**Weekly linting.** A deeper analysis pass checks for orphaned articles (no
incoming backlinks — potentially disconnected knowledge), stale content
(articles not updated despite related schema'd state changes), contradictions
between wiki articles and schema'd records, and style consistency across
articles.

**Graduation reviews.** When ephemeral project vaults approach archival, a
compilation agent identifies knowledge worth graduating to permanent vaults —
lessons learned, vendor evaluations, technology assessments — and prepares
draft articles for human review before graduation.


---


## How it connects to the rest of PuranOS

The Knowledge Wiki is not a standalone system. It is wired into the operating
model at specific integration points.

**Communication agent feeds transcripts.** When the communication agent
processes a meeting email or calendar transcript, it routes the raw content to
the appropriate vault's inbox for compilation. The wiki accumulates project
context without anyone manually writing notes.

**Admin daily briefing includes wiki highlights.** The admin persona's daily
briefing includes a section on wiki activity — new articles compiled, articles
flagged by linting, knowledge graduated from project vaults. This keeps
leadership aware of what the institutional memory is learning.

**Engineering agents query before sizing.** Before starting a design
calculation, engineering personas check the wiki for project-specific
constraints, lessons from similar past work, and vendor-specific notes that
would not appear in schema'd records. The wiki provides the qualitative context
that the quantitative tools need.

**Procurement agents check vendor intelligence.** Before evaluating a vendor
quote, the procurement persona queries the wiki for past experience with that
vendor — delivery reliability, quality issues, negotiation patterns. This
context lives in the wiki because it is synthesized judgment, not a structured
record.

**Sales agents access competitive context.** The commercial vault provides
competitive intelligence — how competitors position, where they have won or
lost, what their technical limitations are. This context informs proposal
strategy and client conversations.


---


## What this is not

The Knowledge Wiki does not replace schema'd state. Equipment specifications,
stream compositions, vendor quotes, work package status — these remain in their
respective typed databases. The wiki holds knowledge that does not reduce to
typed fields and known relationships.

It is also not a general-purpose document store. Documents with compliance or
contractual significance — permits, signed contracts, certified drawings — go
into Paperless-NGX where they receive document IDs and can be referenced from
schema'd records. The wiki holds synthesized knowledge, not source documents.

And it is not a vector store with a UI. The wiki's value comes from LLM
compilation — the active synthesis of raw sources into linked, structured
articles — not from embedding and retrieval. The articles are meant to be read
as coherent narratives, not chunked and similarity-searched.


---


## Further reading

- [Schema Over Memory](schema-over-memory.md) — why schema'd state handles canonical objects
- [Coordination Substrate](coordination-substrate.md) — how OpenProject handles task state
- [Architecture Overview](../architecture/README.md) — where the knowledge layer fits
- [MCP Servers](../architecture/mcp-servers/README.md) — mcpvault in the server listing
