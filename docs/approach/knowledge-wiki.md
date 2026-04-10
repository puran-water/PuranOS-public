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

The Knowledge Wiki closes this gap. It is a file-backed markdown vault
(Obsidian-compatible format) exposed to agents through the `wiki-graph` MCP
server, with a disciplined two-flow workflow: agents drop raw sources into
dated channel folders, and a dedicated knowledgebase agent synthesizes them
into linked compiled articles on a nightly schedule. Graph traversal,
backlinks, fragment reads, and an append-only operations log give agents a
navigable, interlinked knowledge base — without the maintenance burden of a
custom knowledge graph or the retrieval noise of a vector store.

The inspiration is Karpathy's LLM Knowledge Base pattern: use LLMs not just to
retrieve information but to compile and maintain a structured knowledge
repository from raw sources, producing artifacts that are more useful than the
originals.


---


## The pattern: raw sources to linked knowledge

The wiki operates through a defined pipeline with strict write boundaries, not
ad-hoc note-taking.

```
Raw sources (transcripts, emails, chats, calls, web clips, extracted files)
            ↓
Drop flow (any agent writes to raw/<channel>/ with frontmatter)
            ↓
Nightly reconcile (knowledgebase agent synthesizes raw/ → compiled/<category>/)
            ↓
Linked compiled articles (markdown + [[wikilinks]] + sources: frontmatter)
            ↓
Agent retrieval (search_traverse from compiled/_index.md, fragment reads)
            ↓
Weekly lint (broken links, orphans, stale pages, hub detection)
```

### Drop flow

Any agent capturing an inbound corporate comm — a Teams meeting transcript, an
email thread, a WhatsApp alert, a vendor call, a web clip, a digest memo —
writes to `raw/<channel>/<YYYY-MM-DD>-<slug>.md` with typed frontmatter (title,
source, date, actor, refs, tags). Drop-flow agents never touch `compiled/`,
`compiled/_index.md`, or `compiled/_meta/`. The channel taxonomy covers ten
sources: `meetings/`, `email/`, `teams-chat/`, `whatsapp/`, `calls/`, `web/`,
`memos/`, `operations/`, `files/`, `assets/`.

### Reconcile flow

The knowledgebase agent is the only agent that writes to `compiled/`. On a
nightly schedule (2 AM ET), it reads uncompiled raw sources, uses
`search_traverse` to find related compiled articles, and either updates an
existing article or creates a new one under the appropriate category —
`concepts/`, `people/`, `projects/`, `decisions/`, or `glossary/`. Each
compiled article carries `sources: [[raw/...]]` frontmatter pointing back to
the raw drops it was built from, making provenance traceable in both
directions. Batches are bounded (20-30 raws per run); the rest are deferred
rather than rushed.

### Entity resolution for ASR variants

A disciplined resolution check runs before creating any new compiled page.
Teams auto-transcription and other speech-to-text pipelines routinely mangle
proper nouns — *Ensaras* → *Answerus*, *Liberin* → *Libran*, *Burckhardt* →
*Burkhard*. The reconcile flow treats these as variants of the same entity,
never as new entities, and records the alias in a `canonical_aliases:`
frontmatter field on the canonical page. This is LLM-driven per reconcile, not
a hardcoded lookup table: contextual reasoning scales, hardcoded tables rot.

### Output filing for compounding knowledge

When an agent produces a query-derived artifact — a research report, a Marp
deck, a comparison table, a decision memo — it saves the artifact under
`output/` and creates or updates a `compiled/<category>/<slug>.md` summary
that wikilinks back to the artifact. Explorations compound in the graph: a
comparison asked for today becomes a compiled page tomorrow.

### Append-only operations log

Every reconcile, lint, output-filing, and anomaly event appends an entry to
`compiled/_meta/log.md` with a timestamped prefix and a short ops vocabulary
(`reconcile`, `lint`, `file-output`, `skip`, `defer`, `correction`,
`contradiction`, `fix-frontmatter`, `anomaly`, `migration`). The log is
parseable with a single `grep` and gives any agent picking up work a
continuous thread of what the wiki has been doing.

### Weekly lint

Once a week the knowledgebase agent runs a health pass: `unresolved_links()`
for broken `[[wikilinks]]`, `graph_statistics()` for connectivity metrics,
frontmatter date comparisons for stale compiled pages, `backlinks()` checks
for orphan articles, and frequency analysis for terms mentioned in many
articles without a dedicated page. Findings land in a dated lint report and a
summary entry in `log.md`.


---


## Relationship to Paperless-NGX

The boundary between Paperless-NGX and the Knowledge Wiki is deliberate and
load-bearing.

| Concern | Paperless-NGX | Knowledge Wiki |
|---------|---------------|----------------|
| What it holds | Documents needing a document ID | Synthesized knowledge articles and raw corporate comms |
| Examples | Permits, vendor submittals, contracts, P&IDs, certificates | Meeting synthesis, email threads, design rationale, competitive intel, lessons learned |
| Primary consumer | Cross-system reference (linked from schema'd records) | Agent context for synthesis and decision support |
| Identity model | Document ID, tags, correspondent, document type | Wiki article path, `[[wikilinks]]`, `sources:` provenance chains |
| Retrieval pattern | By document ID or structured search | `search_traverse` from `compiled/_index.md`, backlink navigation, fragment reads |
| Authorship | Humans upload; OCR extracts | Agents drop and compile from raw sources |

A vendor submittal goes into Paperless because it needs a document ID that the
procurement schema can reference. The lessons learned from reviewing that
submittal — what was strong, what was missing, how it compared to competitors
— go into the wiki as a compiled page under `decisions/` or `projects/`, with
its raw sources under `raw/memos/` or `raw/meetings/` and the Paperless doc ID
referenced in frontmatter.

The two systems complement each other. Paperless is the filing cabinet. The
wiki is the institutional memory.


---


## Scoped corpora when you need them

The default is a single shared `professional` vault. When a specific project
or asset needs a hard-isolated retrieval surface — for example, a materialized
tender package on a bid, or an archived project closeout corpus — optional
scoped MCP servers (`wiki-project`, `wiki-asset`) can be launched against a
materialized `generated_corpora/views/.../vault` tree. These are read-oriented
surfaces for persistent document sets; they do not replace the shared
corporate wiki and they are not where corporate comms accumulate.

The retired three-vault layout (separate ops, engineering, commercial vaults
each on their own port) was consolidated into the single shared vault. One
vault made entity resolution cleaner, eliminated split-graph problems when the
same company appeared in multiple vaults, and simplified the retrieval
contract for every agent.


---


## Implementation: file-backed markdown behind an MCP graph server

```
┌──────────────────────────────────────────────────────────────┐
│  File-backed markdown vault on OCI                           │
│    /home/ubuntu/wiki/professional/                           │
│      raw/<channel>/*.md    (immutable dated captures)        │
│      compiled/<category>/  (LLM-synthesized linked pages)    │
│      compiled/_index.md    (master index)                    │
│      compiled/_meta/log.md (append-only operations log)      │
│      output/               (generated artifacts)             │
│      templates/            (article templates)               │
│                                                              │
│    Obsidian-compatible format — but not a human UI target.  │
├──────────────────────────────────────────────────────────────┤
│  wiki-graph MCP server (fork of mcpvault)                    │
│    15 file I/O tools  + 8 graph traversal tools              │
│    2 health tools     + safe rename-with-link-updates        │
│    MCP resources (graph stats, manifest, orphans, hubs)      │
├──────────────────────────────────────────────────────────────┤
│  Agents                                                      │
│    Drop-flow agents write raw/                               │
│    Knowledgebase agent owns compiled/ and output/            │
│    All other agents read via search_traverse + fragment      │
│    Humans query the wiki by asking an agent, not by          │
│    opening the vault directly.                               │
└──────────────────────────────────────────────────────────────┘
```

### wiki-graph MCP server

`wiki-graph` is a fork of [mcpvault](https://github.com/bitbonsai/mcpvault)
deployed at `/home/ubuntu/wiki/tools/wiki-graph/` on OCI. It runs as a
long-lived STDIO process pointed at the shared vault and exposes three groups
of primitives:

- **File I/O (15 tools)** — `read_note`, `write_note`, `patch_note`,
  `delete_note`, `list_directory`, `search_notes`, `move_note`, `move_file`,
  `read_multiple_notes`, `update_frontmatter`, `get_frontmatter`,
  `get_notes_info`, `manage_tags`, `get_vault_stats`, `list_all_tags`. These
  are the mcpvault primitives for reading and writing markdown files, editing
  YAML frontmatter, and managing tags.

- **Graph traversal (8 tools)** — `resolve_wikilink`, `backlinks`,
  `forwardlinks`, `neighbors`, `traverse`, `find_path`, `search_traverse`,
  `fragment`. `search_traverse` is the key retrieval primitive: it scores
  notes by query relevance while following `[[wikilinks]]` outward from a
  start path (typically `compiled/_index.md`), giving agents graph-walk from
  curated knowledge instead of brittle full-text similarity. `fragment`
  reads a single heading section and cuts token consumption by 60-80%
  compared to reading a whole compiled article.

- **Health and safe edits (3 tools)** — `unresolved_links` and
  `graph_statistics` drive the weekly lint pass; `rename_with_link_updates`
  renames a note and rewrites every inbound wikilink atomically (never use
  `move_note` for renames — it breaks inbound links silently).

The server also exposes **MCP resources** for ambient context: graph
statistics, a note manifest, orphan lists, and hub notes. Agents can pull
these as read-only context without making a tool call, keeping retrieval
cheap when what they need is a quick orientation rather than a targeted
query.

### Why file-backed and not a database

Markdown files on a filesystem are inspectable, version-controllable, and
portable. If the MCP server goes down, the wiki is still a directory of
readable text files. The graph is not materialized in a separate index
process — it is computed from wikilinks in the files themselves — which means
there is no sync layer, no eventual consistency, and no stale-index problem.
Drops become visible to reconcile the moment they land.

### Why not a human UI

Earlier iterations of this architecture exposed Obsidian's desktop app on OCI
as a human browsing surface. That path was retired. Humans interact with the
wiki by asking agents — via email, chat, or an agent session — rather than by
opening the vault directly. The reasons are practical: Obsidian plugins
(Dataview, Templater, Canvas, and friends) were zero ROI against the MCP
retrieval surface agents actually use, maintaining an Obsidian desktop on a
headless CPU-only OCI instance added operational cost with no user on the
other end of it, and the drop/reconcile discipline assumes agents are the
only writers. A concurrent human editor would collide with nightly reconcile
and break the `sources:` provenance chain.

The vault is a backend knowledge store, not a human-facing application. Any
optimization targets the MCP server tools, agent prompts, or vault structure
conventions — never Obsidian UI plugins.


---


## Retrieval hierarchy: write to raw, read from compiled

Raw and compiled are the same vault but different roles. Raw is the immutable
event log — dated, provenance-preserving. Compiled is the aggregated state —
curated, current, wikilinked. Agents use a layered retrieval strategy:

1. **Preferred entry: `search_traverse` from `compiled/_index.md`.** Graph-walk
   from curated knowledge. Compiled pages are the authored answer; treat them
   as the source of truth for "what do we know about X?"
2. **Full-text fallback: `search_notes`.** If compiled has a gap, this searches
   the entire vault. If top hits are `raw/`, that signals an uncompiled gap —
   the knowledge exists but hasn't been synthesized yet. The raw hit is usable
   context; the knowledgebase agent will compile it on the next reconcile.
3. **Provenance drill-down.** From a compiled page, follow the `sources:`
   frontmatter wikilinks DOWN to `raw/` to verify claims before acting on
   high-stakes decisions. `read_multiple_notes` batches the verification.
4. **Recency override.** For events in the last 24-48 hours, check
   `raw/<channel>/` directly — compiled has a nightly reconcile lag and can't
   cover same-day events. This is the one case where any agent reads raw for
   retrieval, not just provenance.
5. **Token efficiency.** Once a compiled page is located, prefer `fragment` on
   the relevant section over a full `read_note`. Compiled pages can be long;
   section reads cut token consumption by 60-80%.

In practice, most agents use step 1 for context gathering, then `fragment` for
the specific section. Steps 3-4 are for the knowledgebase agent during
reconcile or any agent needing same-day events.


---


## How it connects to the rest of PuranOS

The Knowledge Wiki is wired into the operating model at specific integration
points.

**Communication agent feeds raw sources.** When the communication agent
processes a meeting transcript, email thread, or WhatsApp message, it routes
the raw content to the correct `raw/<channel>/` folder with typed frontmatter.
The wiki accumulates project context without anyone manually writing notes.

**Nightly reconcile compiles raw into compiled.** The knowledgebase agent
runs on a 2 AM ET schedule, picks up everything that landed in `raw/` since
the last run, and synthesizes it into compiled articles with wikilinks back
to the raw sources.

**Engineering agents query before sizing.** Before starting a design
calculation, engineering personas run `search_traverse` from
`compiled/_index.md` to pull project-specific constraints, lessons from
similar past work, and vendor-specific notes that would not appear in
schema'd records. The wiki provides the qualitative context that the
quantitative tools need.

**Procurement agents check vendor intelligence.** Before evaluating a vendor
quote, the procurement persona queries the wiki for past experience with
that vendor — delivery reliability, quality issues, negotiation patterns.
This context lives in the wiki because it is synthesized judgment, not a
structured record.

**Sales agents access competitive context.** Competitive intelligence,
positioning notes, and market research compile into `concepts/` and
`projects/` pages that sales personas read during proposal work.

**Output filing closes the loop.** When an agent produces a research report,
a comparison table, or a decision memo, it files the artifact under
`output/` and summarizes it as a new or updated compiled page. Today's
one-off exploration becomes tomorrow's reusable knowledge.


---


## What this is not

The Knowledge Wiki does not replace schema'd state. Equipment specifications,
stream compositions, vendor quotes, work package status — these remain in
their respective typed databases. The wiki holds knowledge that does not
reduce to typed fields and known relationships. It also never duplicates
data that a structured MCP server already owns: twenty-crm holds sales deal
notes, openproject-mcp holds work package state, paperless-ngx holds OCR'd
document content, compliance DB holds regulatory calcs. The wiki exists for
unstructured sources without an MCP alternative and for cross-source narrative
synthesis the MCPs can't produce — never as a second source of truth for
structured state.

It is also not a general-purpose document store. Documents with compliance or
contractual significance — permits, signed contracts, certified drawings —
go into Paperless-NGX where they receive document IDs and can be referenced
from schema'd records.

And it is not a vector store with a UI. The wiki's value comes from LLM
compilation — the active synthesis of raw sources into linked, structured
articles — not from embedding and retrieval. The articles are meant to be
read as coherent narratives, not chunked and similarity-searched.

It is also not a human-facing application. Humans query the wiki through
agent sessions; the vault is a backend store for agents to read and write.


---


## Further reading

- [Schema Over Memory](schema-over-memory.md) — why schema'd state handles canonical objects
- [Coordination Substrate](coordination-substrate.md) — how OpenProject handles task state
- [Architecture Overview](../architecture/README.md) — where the knowledge layer fits
- [MCP Servers](../architecture/mcp-servers/README.md) — wiki-graph in the server listing
