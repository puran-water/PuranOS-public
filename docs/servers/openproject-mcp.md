# OpenProject MCP Server

`openproject-mcp` is the OpenProject integration layer for PuranOS. It turns OpenProject into a usable collaboration surface for both humans and agents without forcing the rest of the system to speak raw API v3.

## Why This Exists

OpenProject is valuable in this repo for more than ticket storage. It is the shared system of record for:

- project tasks
- dependencies
- comments
- hierarchy
- approvals
- agent-visible state

An MCP server is the right abstraction because it keeps those operations typed, testable, and reusable across personas and runtimes.

## What The Server Does

### Standard OpenProject Operations

- project discovery
- work package CRUD
- user and membership lookup
- comments and updates
- relations and hierarchy
- statuses, priorities, and views
- schema and form inspection

### Agent-Collaboration Operations

The server now also exposes the OpenProject-specific collaboration surface needed for agent workflows:

- agent task creation
- agent task retrieval and listing
- explicit agent state updates
- result posting
- completion flows
- typed custom-field integration for agent state

## Typed Agent State

The current design stores agent-task state in OpenProject custom fields rather than hiding orchestration metadata in the description body for new tasks.

Logical fields include:

- `agent_owner`
- `structured_input`
- `reply_actor`
- `allowed_side_effects`
- `approval_state`
- `workflow_state`
- `blocked_reason`
- `case_id`
- `state_version`
- `last_agent_summary`
- `last_agent_run_at`

Legacy description metadata remains readable as a migration fallback, but new typed-task flows are built around custom fields.

## Why This Matters

This server is one of the key reasons a full OpenProject fork is unnecessary right now.

With:

- schema/form APIs
- custom fields
- comments and relations
- views and work package hierarchy
- native outgoing webhooks

OpenProject already provides the collaboration substrate the repo needs. The MCP layer turns that substrate into a tool surface agents can use reliably.

## Important Tool Families

### Integration

- connection testing
- schema and form inspection
- agent-state config inspection

### Work Packages

- generic create, update, get, list, delete
- comment and relation operations
- agent-task lifecycle operations

### Collaboration Helpers

- custom field mapping
- assignee resolution
- typed field serialization

## Configuration

The internal operational repo also carries a tracked OpenProject MCP environment example used to document the expected runtime contract.

Important environment variables:

- `OPENPROJECT_URL`
- `OPENPROJECT_API_KEY`
- `OPENPROJECT_HOST_HEADER`
- `OPENPROJECT_FORWARDED_PROTO`
- `OPENPROJECT_AGENT_FIELD_MAP_JSON`

The host header and forwarded proto settings are useful when talking to a private origin through a local tunnel while preserving the application's canonical-host expectations.

## Operational Role In The Repo

`openproject-mcp` is the bridge between:

- the project board humans use
- the orchestration service that dispatches and audits work
- the project-management persona that needs a rich task-management tool surface

Without it, agents would either need brittle raw API wrappers in prompts or a much heavier OpenProject customization strategy.

## Related Docs

- [Agent Runtime Architecture](../architecture/agent-runtime/README.md)
- [OpenProject Service Notes](../services/openproject.md)
