# Specialist roles and handoffs

The owner coordinates PuranOS work. `pe-process` runs `/proposal-workflow` and `/bep-workflow`, drawing on the shared `/sizing-to-balance` engineering package. The retired `pe-lead` persona is not a current dispatch target.

## Engineering

| Persona | Responsibility |
| --- | --- |
| `pe-process` | Process duties, heuristic sizing, engine-driven simulation and whole-plant balance. |
| `pe-cad` | Process drawings, equipment geometry, site layout and coordinated visual review. |
| `pe-civil` | Terrain, grading, drainage, buried networks, hydraulic profile and civil quantities. |
| `pe-piping` | Spec-driven 3D routing, slope and access checks, and piping material take-offs. |
| `pe-mechanical` | Pumps, piping hydraulics, heat transfer, mixing, corrosion and equipment mechanics. |
| `pe-structural` | Analytical models and structural design checks for the consulting engineer to review. |
| `pe-electrical` | Power distribution, loads, cables, protection and instrumentation engineering. |
| `pe-automation` | PLC and SCADA authoring and offline verification from accepted requirements. |
| `pe-ehs` | Plant hazards, mitigation hierarchy and discipline-specific follow-up. |
| `pe-water-supply` | Municipal treatment, distribution, transmission corridors and pressure zoning. |
| `pe-separation` | Membrane, resin and separation advice, vendor evidence and pilot interpretation. |
| `pe-narrative` | Prose in named slots from assembled facts; process engineering owns technical review. |
| `pe-deliverables` | Coordinated datasheets, schedules and document packages from canonical records. |
Process simulation on the proposal path belongs to `pe-process`; separation advice does not create a second sizing authority. Civil owns site and hydraulic-profile work; piping owns routing; CAD realizes and presents their accepted geometry. Electrical engineering supplies requirements to automation. EHS flags risks and routes mitigations to the discipline that can resolve them. Narrative drafting does not approve engineering or invent missing calculations.

## Commercial, delivery and operations

- `cost-estimator` and `procurement`: scope-backed estimates, supplier evidence and purchasing preparation. Commercial commitments require authorized action.
- `project-management`: project state, dependencies and delivery coordination in the relevant business system.
- `project-finance`: explicit financial assumptions, scenarios and model evidence.
- `maintenance` and `compliance`: equipment history, operating investigations and sourced regulatory work.
- `knowledgebase`: retained unstructured context linked to the authoritative records.
- `pw-admin`, `pw-bookkeeper` and `pw-leadgen`: Puran Water's administrative, accounting and origination scope. Portfolio-company roles have separate systems and permissions.
- `mc-lead`, `mc-story`, `mc-visual`, `mc-audio` and `mc-review`: evidence, direction, visuals, audio and independent review for technical communication.

A configured persona is a responsibility boundary, not a claim that every tool is deployed or that a plugin is available for public installation. Public MCP packages can run in a user's chosen client; internal workflow execution and project access require a separate arrangement. A caller's model does not gain approval authority by adopting a role's instructions.

[Workflow](../../workflows.md) · [Studio](../../studio.md) · [Public packages](../../public-tools.md)
