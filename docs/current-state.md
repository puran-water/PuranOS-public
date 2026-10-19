# Current capability notes

Reviewed October 19, 2026 against the [source snapshot](source-snapshot.json).

| Surface | What the reviewed implementation supports | Boundary |
| --- | --- | --- |
| Engineering calculations | Shared sizing and process-model workflows, retained inputs, result artifacts, run receipts and series code pins. | Applicability and convergence are examined for each case. Dependency changes are reported; a prompt is not a deterministic specification. |
| Equipment and CAD | Stable equipment identities, registry-bound geometry, typed access/clearance checks and model-derived deliverables. | A tag match or generated drawing alone does not establish an acceptable design. |
| Engineering packages | Shared records feed schedules and document renderers; document identity, revision and review status accompany the package. | Required checks and release authority are workflow-specific. |
| Finance | Archived model inputs and projections, scenario comparisons and structured actual-versus-projection analysis. | Account mappings, periods, signs and assumptions require review. There is no claim of automatic end-to-end reconciliation with all plant telemetry. |
| Operations | Configured signal review, equipment history and model-supported investigation. | Diagnostic coverage is limited to connected data and implemented models; covered diagnostic escalation is in pilot. The diagnostic workflow is advisory. |
| Customer interoperability | Structured deliverable information, published reference schemas and separately authorized project interfaces. | No universal compatibility or default customer access to internal tools. |
| Public MCP | Live service discovery, fit checks and input checklists at the published endpoint. | No private project records or engineering simulations. Optional submission is consent-based and separate from discovery. |
| Reusable learning | Reviewed corrections committed to shared tools, workflows and instructions. | Adoption into an existing design is an explicit revision/replay decision, not automatic model learning. |

## What changed since the earlier public overview

The earlier overview emphasized a broad enterprise stack and fixed counts of agents, servers and schemas. The current guide follows the engineering work: source basis, specialist responsibilities, model execution, connected records, deliverable generation and review.

The refreshed documentation includes CAD identity and clearance workflows, engineering-series reproducibility controls, shared deliverable artifacts, finance comparison boundaries and the public MCP connection. It removes unsupported fleet-wide guarantees and avoids counts that become stale as the monorepo changes.

## Publication boundaries

The public repository remains documentation-only. Source paths in the manifest identify evidence in the private monorepo; they are not public downloads. Client documents, database contents, credentials, private endpoints, correspondence and infrastructure configuration are excluded.

The website’s schema catalog remains explicitly dated July 15, 2026. This documentation refresh does not relabel that historical export as a current database snapshot. Validate an actual integration against the schema and interface agreed for the project.
