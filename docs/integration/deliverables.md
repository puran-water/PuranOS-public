# Deliverables for people and agents

Puran Water’s engineering delivery is designed to support human review and continued use by a customer’s tools. PuranOS workflows retain structured engineering records and produce human-facing files from that information.

## What accompanies the output

The actual package depends on the engagement. Define:

- document identity, revision, status and superseded revision;
- stable equipment identifiers and displayed tags;
- values with explicit units and an identifiable source basis;
- assumptions, model references and unresolved questions;
- preparation, checking and approval information;
- artifact metadata, including content hashes where supplied;
- the agreed export schema or authorized interface.

Shared deliverable helpers in the monorepo support rendered XLSX, SVG and PDF artifacts, with canonical payload metadata. Engineering-package workflows also support document rendering from retained source information. These are implementation capabilities, not a promise that every file type automatically contains every possible field.

## A paired example

The [website demonstration](https://revamp.puranwater-com.pages.dev/puranos/deliverables/) renders a pump information sheet and the corresponding [JSON example](https://revamp.puranwater-com.pages.dev/examples/equipment-handoff.json) from the same illustrative record. The record preserves the earlier head value, the revision reason and the open review item.

This is demonstration data, not an issued design or a production API schema. It shows the integration pattern: a customer agent can examine the same information a person sees, including its unresolved status.

## Customer uses

A scoped agent can compare a supplier response against a requirement, identify changes between revisions, collect equipment information for maintenance or retrieve the assumptions behind a calculation. These uses reduce repeated extraction and transcription; quantified time or cost benefits require measurement for the actual workflow.

## Define the interface

Agree the project identifiers, schema/version, export formats, authentication, allowed operations and revision behavior. A delivered snapshot and a live query are different contracts. Keep access scoped to the customer’s authorized project data.

The [public MCP connection](mcp.md) provides service discovery and preparation tools. It does not expose these private project records. The [published schema catalog](https://puranwater.com/puranos/schemas/) is a dated reference, so integration compatibility must be checked against the agreed contract.

## Financial information

Retain the original model baseline when generating a revised forecast. Actual-versus-projection comparisons require explicit period alignment, account mapping, units and sign conventions. A reviewed update creates a new basis; it does not rewrite the old investment case or establish an automatic design-to-telemetry-to-accounting connection.
