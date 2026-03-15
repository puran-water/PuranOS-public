# Agent Runtime

The agent runtime is the control plane for this repo. It turns inbound events into governed work, keeps execution durable, and makes collaboration visible across both humans and agents.

## Why This Exists

Most agent demos treat every prompt like an isolated interaction. That is not enough for enterprise work.

Real delivery workflows need:

- durable ingress deduplication
- resumable execution
- predecessor and approval gates
- auditable side effects
- a shared board that humans can understand

The runtime in this repo is designed around those requirements.

## Core Components

### Communication Agent

`agents/communication-agent/` is the long-running ingress and orchestration service. It handles:

- Microsoft Graph email, calendar, and transcript events
- OpenProject native outbound webhooks
- fallback polling for tasks that become runnable without producing a fresh task event
- scheduled work

The runtime now also includes:

- declarative schedule loading from `agents/schedules.yaml`
- prompt-template-backed recurring jobs
- dynamic specialist discovery from `agent-configs/`
- explicit policy shaping passed to downstream mutation surfaces

### Postgres Execution Ledger

The orchestration database stores:

- `work_items`
- `conversation_sessions`
- `activity_log`
- `ingest_events`
- `work_item_attempts`
- `state_transitions`
- `side_effect_log`
- `execution_leases`

These tables exist to make the runtime reliable, not merely observable.

The shared Postgres stack now provisions orchestration as a first-class database alongside application databases.

### OpenProject Collaboration Surface

OpenProject is used as the shared board for long-running work. In the current implementation it carries:

- task hierarchy and relations
- predecessor gating
- comments and internal comments
- assignee-driven routing
- typed custom fields for agent state
- native outgoing webhooks

### Office Policy Bridge

The Office MCP integration enforces policy where mutations actually occur. That means reply authority and allowed side effects are checked at execution time rather than treated as advisory prompt text.

### Persona Discovery and Skill Packaging

The runtime no longer depends on a fixed hardcoded specialist roster. It can discover persona configs from the repo and combine them with persona-local shared skill overlays, including shared email-formatting assets.

## Current State Model

### OpenProject Stores Collaborative State

Typed OpenProject custom fields hold agent-visible task state such as:

- agent owner
- structured input
- reply actor
- allowed side effects
- approval state
- workflow state
- blocked reason
- case identifier
- state version
- last agent summary
- last agent run timestamp

This makes project state inspectable in the same system where humans already manage work.

### Postgres Stores Execution State

Postgres is authoritative for:

- ingress idempotency
- attempt history
- lease ownership
- retry semantics
- state transition history
- side-effect audit records

This is the layer that protects the system from duplicate webhook deliveries, restart ambiguity, and invisible failure modes.

## Webhook-First With Poller Fallback

The repo now runs OpenProject in webhook-preferred mode, but the fallback poller remains by design.

That is not because webhooks are insufficiently fast. It is because correctness sometimes requires a background sweep:

- task `B` may be blocked by predecessor `A`
- OpenProject emits an event for `A` when `A` is completed
- `B` may become runnable without receiving its own fresh task event

The poller exists to discover and dispatch those newly-unblocked tasks within a bounded delay.

## Runtime Flow

```text
External event
    ↓
Ingress validation and deduplication
    ↓
Work item creation / correlation
    ↓
Lease acquisition and attempt record
    ↓
Policy and gate evaluation
    ↓
Specialist execution through persona-specific tools
    ↓
Side-effect audit + result persistence
    ↓
Human-visible project update
```

## Why This Design Has High ROI

This architecture gives the system both of the things enterprise AI efforts usually struggle to combine:

- reliable execution semantics
- understandable project state

It avoids trying to turn a project board into a job queue, and it avoids hiding all meaningful collaboration state inside an internal database.

## Related Docs

- [Communication Agent README](../../runtime/communication-agent/README.md)
- [OpenProject MCP README](../../servers/openproject-mcp.md)
- [OpenProject Service Notes](../../services/openproject.md)
