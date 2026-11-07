# MCP surfaces and engineering libraries

MCP is the agent-facing interface to selected calculations and domain operations. Shared Python code imports engineering libraries directly; it does not call another MCP server to obtain a local library function.

## Public calculations

[Current public packages](../../public-tools.md) cover fluids, heat transfer, water chemistry, corrosion, dynamic QSDsan studies and steady-state WaterTAP flowsheets. They run through local stdio and use a public support package. Each repository documents installation, limitations and licensing.

These tools return structured numerical results, diagnostics and provenance appropriate to their operation. The caller supplies interpretation and reporting. Public availability does not imply complete qualification of every correlation or platform.

## Internal workflows and records

The complete proposal/BEP procedures, project registry, procurement records, vendor pricing and approval operations are separate internal capabilities. A public tool installation does not grant access to them. Studio publishes retained review views under project access controls.

The public intake connector described in [the integration guide](../../integration/mcp.md) is a distinct discovery and engagement surface. It does not expose the internal engineering fleet.

## Source and updates

The monorepo is the source of truth for the released engineering servers. Its one-way mirror reconstructs the permitted export, checks privacy, secrets, rights and package boundaries, then advances each public branch. Version tags remain immutable. A failed scan refuses publication without rolling back the internal landing.

[Workflows](../../workflows.md) · [Studio](../../studio.md) · [Specialists](../personas/README.md)
