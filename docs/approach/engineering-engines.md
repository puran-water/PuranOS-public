# Engineering computation

PuranOS connects agents to engineering methods through shared tools, model contracts and retained results. The objective is an inspectable calculation that can be carried into a design decision.

## Models and methods

The reviewed monorepo includes WaterTAP and QSDsan workflows, chemical/speciation tooling, hydraulic and thermal calculations, shared sizing utilities and process-model contracts. Engine selection depends on the process and the model’s implemented range; the presence of a simulator does not establish suitability for every feed or operating state.

Shared process-model work describes streams, component bases and transformations. Whole-plant calculations must account for connections and recycles. A changed arrangement requires the affected balances to be evaluated on the changed basis.

## Source characterization

Keep measured values, literature ranges and assumptions identifiable. Unit conversions and component mapping are engineering transformations with their own basis. Do not interpret a literature estimate as a site measurement or a successful schema validation as a verified feed characterization.

## Calculation and document handoff

Retained sizing and simulation results support equipment requirements, schedules and engineering documents. Shared renderers read the retained information. A result should identify its inputs, units, method, software/model revision and known limits sufficiently for the reviewer to inspect it.

The equipment required by the process and the equipment offered or installed are separate records. A comparison can reveal a gap; it does not establish acceptance by itself.

## Reproducibility

The calculation runner retains stage history and can require replay when inputs change. Engineering series code pins and library-version receipts help explain the implementation behind a result. The same natural-language prompt can still produce different assumptions or tool arguments.

Numerical convergence, conservation checks and software tests answer specific questions. They do not replace engineering judgment or demonstrate installed plant performance.

## From operation back to design

Configured operating evidence can support a process investigation. A discrepancy may indicate a measurement issue, an unsuitable parameter, a missing mechanism or a real process change. Engineers review the evidence before updating a shared model or a subsequent design.

[Execution controls](../architecture/agent-runtime/README.md) · [Structured deliverables](../integration/deliverables.md)
