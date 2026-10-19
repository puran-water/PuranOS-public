# Identity and connected records

Engineering continuity depends on knowing which item a record refers to and which revision of the basis produced it.

## Equipment position and physical asset

An equipment position represents a function in the plant. An installed asset represents a physical item, potentially with a serial number and installation history. A displayed tag is a useful label; it should not be the only way systems establish identity.

The equipment-identity domain and CAD workflows support stable bindings between engineering records and placed objects. Related procurement and maintenance records can refer to that identity. Each relationship requires an actual declared mapping; similar text does not prove that two records describe the same item.

## Three complementary sources of structure

1. Domain/application schemas describe operational and business records.
2. Shared engineering contracts describe information passed between calculations and tools.
3. Project artifacts retain results, document identity and source context.

Narrative knowledge preserves rationale that does not fit a field. It complements the structured basis instead of silently overriding an approved requirement.

## Read paths and write ownership

Ontology tools retrieve records and traverse declared relationships. Domain tools own their mutation semantics. Project scope, lifecycle conditions and audit context must be checked on the actual write surface.

Client-facing views and interfaces are deployment-specific. The public website’s [schema catalog](https://puranwater.com/puranos/schemas/) is a dated reference export, not a live view of customer records. Authentication and authorized project scope are specified separately.

## Handoff consequence

A stable equipment identity lets a customer compare a requirement, drawing and supplier response. Revision metadata lets the customer identify which basis applies. Neither removes the need to review engineering suitability.

[Deliverable integration](../../integration/deliverables.md) · [Current capability notes](../../current-state.md)
