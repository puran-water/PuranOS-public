# Communication Agent

The communication-agent is the repo's long-running orchestration service. It connects inbound events to the right specialist persona, persists execution state durably, and updates the shared project system so human and agent work remain aligned.

## Why This Exists

Most organizations do not need an agent that can do everything. They need a control plane that can:

- receive work from real systems
- route it to the correct specialist
- respect approvals and reply authority
- survive retries, duplicates, and restarts
- leave behind an auditable record

That is what this service provides.

## What It Handles Today

### Microsoft Graph

- inbound email
- calendar-related events
- meeting transcript ingestion
- subscription lifecycle management

### OpenProject

- native outgoing webhooks for work package and comment events
- fallback polling for tasks that become runnable without a fresh task event
- agent-task claim, dispatch, result posting, and completion

### Scheduled Work

- scheduled briefings and recurring operational workflows
- attempt and lease tracking for scheduled execution paths

## Architecture

```text
External event
    ↓
FastAPI ingress
    ↓
Validation + durable deduplication
    ↓
Work item correlation in Postgres
    ↓
Lease + attempt record
    ↓
Policy / approval / predecessor checks
    ↓
Persona execution
    ↓
Side-effect audit + result persistence
    ↓
Human-visible update in OpenProject or the source system
```

## State Model

The service is intentionally hybrid.

### OpenProject Stores Long-Running Collaborative State

OpenProject carries:

- task hierarchy
- predecessors and relations
- comments and internal comments
- assignee-driven routing
- typed agent-state fields

This is the state humans should see and work with.

### Postgres Stores Execution Reliability State

The orchestration database carries:

- work items
- conversation sessions
- activity log
- ingress deduplication records
- attempt history
- state transitions
- side-effect audit records
- execution leases

This is the state the runtime needs to behave correctly.

## Current Reliability Features

- webhook ingress deduplication
- resumable execution via attempts and leases
- side-effect logging
- reply-actor and mutation-policy enforcement
- OpenProject webhook-preferred processing with poller fallback
- typed OpenProject custom-field state instead of hidden description metadata for new tasks

## Why Webhook-Preferred Still Keeps A Poller

The fallback poller is not a legacy artifact. It remains necessary for correctness.

Example:

- task `B` is blocked on predecessor `A`
- a human completes `A`
- `B` becomes runnable
- `B` may not receive a fresh task event of its own

The fallback sweep exists to discover that newly-runnable work within a bounded delay.

## Key Modules

| Module | Purpose |
|--------|---------|
| `src/main.py` | FastAPI ingress, webhook handlers, scheduler bootstrap, OpenProject webhook path |
| `src/op_poller.py` | fallback sweep, gating logic, OpenProject claim and dispatch flow |
| `src/session_store.py` | Postgres-backed persistence for work items, attempts, transitions, side effects, and leases |
| `src/specialist_caller.py` | persona execution via the local agent runtime |
| `src/codex_specialist_caller.py` | alternate execution path for Codex-based specialist invocation |
| `src/policy.py` | policy shaping passed into downstream tool layers |
| `src/webhook/` | source-specific webhook handling |
| `src/subscriptions/` | Microsoft Graph subscription lifecycle |

## Configuration Surfaces

Documented configuration surfaces:

- communication-agent environment example in the internal operational repo
- OpenProject custom-field manifest
- OpenProject outgoing webhook manifest

Important runtime contracts:

- OpenProject typed agent-state field mapping
- OpenProject webhook secret
- Postgres connection string
- Graph client credentials
- persona mailbox map
- allowed sender and reply authority configuration

## Deployment Model

The service is intended to run as a long-lived process under systemd or an equivalent supervisor.

Recommended operating pattern:

1. run service-to-service traffic against internal origins when possible
2. keep webhook secrets and tokens out of tracked files
3. provision OpenProject custom fields and outgoing webhook definitions from the repo manifests
4. keep webhook-preferred mode enabled and the poller configured as fallback

## Operational Value

This service is where the repo becomes more than a collection of tools.

It gives the system:

- a durable execution ledger
- real task collaboration with humans
- governed side effects
- a practical path from inbound request to completed project work

## Related Docs

- [Documentation Index](INDEX.md)
- [Automation Architecture](AUTOMATION.md)
- [Consolidation Notes](CONSOLIDATION.md)
- [Agent Runtime Architecture](../../architecture/agent-runtime/README.md)
