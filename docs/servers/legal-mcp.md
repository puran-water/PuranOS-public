# Legal MCP Server

`legal-mcp` is the in-house-counsel contract-twin surface for PuranOS. It models contracts, their clauses, the flowdown links that propagate obligations from prime agreements to subcontracts, and the typed legal facts (LD caps, cure periods, limitation-of-liability carve-outs, precedence terms) that govern how those contracts behave. On top of that model it runs a deterministic scenario engine that answers contract questions with reproducible, audited results. The server writes to a dedicated `legal` Postgres database; reads across domains are served through `ontology-mcp`.

## Why This Exists

A design-build-operate firm signs and inherits a large number of overlapping contracts — owner agreements, EPC and DBIA forms, vendor purchase orders, subcontracts, and the flowdown obligations between them. The questions that matter at delivery time are not "what does the contract say" in the abstract but "given these clauses, what is the applicable LD cap on this term," "which document controls when two conflict," "is this vendor delay excusable and is the cure period still open," and "does the limitation of liability carve out this category of claim." Answering those questions by re-reading PDFs every time is slow, inconsistent, and unauditable.

`legal-mcp` makes the contract a structured, queryable twin. Clauses and legal facts become typed rows; obligation flowdown becomes explicit links; and contract questions become deterministic scenario runs whose inputs, logic path, and conclusion are recorded. This gives the legal persona the same schema'd-state discipline the engineering and commercial domains already have.

## What The Server Does

### Contract and clause modeling

- Register a contract and decompose it into clauses, each carrying its source location and classification.
- Record typed **legal facts** against clauses: LD caps (per-term and aggregate), cure periods, excusability conditions, precedence/order-of-precedence terms, and limitation-of-liability carve-outs.
- Link **flowdown** relationships so that an obligation in a prime agreement is traced to the subcontract clauses that inherit it.

### Deterministic scenario engine

A reproducible evaluation layer that answers contract questions against the persisted facts rather than against free-form prose:

- **LD caps** — applies per-term and aggregate liquidated-damages caps, honoring the most specific applicable cap.
- **Precedence / later-in-time** — resolves conflicts between documents using the contract's stated order of precedence, falling back to later-in-time where precedence is silent.
- **Vendor cure period and excusability** — determines whether a delay or default is excusable and whether the cure window is still open as of a given date.
- **Limitation-of-liability carve-outs** — evaluates whether a claim category escapes the LoL cap via an enumerated carve-out.

Each evaluation is persisted as an append-only scenario run with a decision trace recording the inputs considered, the clauses and facts consulted, the logic path taken, and the conclusion reached.

### Portfolio queries

- `portfolio_query` surfaces upcoming deadlines (cure windows, notice periods, milestone obligations) across the contract portfolio, so the legal persona can act before a window closes.

## Append-Only Provenance

Every write to the `legal` database is append-only and carries `reviewed_by` provenance — facts, flowdown links, and scenario runs are never silently overwritten. Append-only triggers are owned at the database level so the runtime role cannot disable them. The combination of typed facts, decision traces on every scenario run, and reviewer attribution gives the contract twin the audit trail that contract disputes and counsel review require.

## Governed Actions

`legal-mcp` contributes governed mutations to the ontology action catalog, declared in its domain-local `ontology.actions.yaml`: `ingest_contract_document`, `ingest_clause`, `ingest_verified_clause_atom`, `ingest_flowdown_link`, `mark_verification`, `run_scenario`, `supersede_clause`, `supersede_fact`, and `supersede_flowdown_link`. Each specifies inputs, preconditions referencing lifecycle state, effects, and a `subjects` list, and emits an append-only action event when executed.

## Cross-Reference

- `ontology-mcp` — the universal read layer that resolves cross-domain relationships into and out of the `legal` database; domain reads happen there, not in `legal-mcp`.
- `contractor-management-mcp` — the commercial sidecar for DBIA contractor/subcontractor management (SOV lines, pay applications, change orders, claims). `legal-mcp` owns the contract *terms* and their interpretation; `contractor-management-mcp` owns the contract *money*. The boundary is clean.
