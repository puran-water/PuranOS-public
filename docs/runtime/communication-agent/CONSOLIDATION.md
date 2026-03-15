# Communication Agent Consolidation Notes

This service represents a deliberate consolidation of ingress, routing, and orchestration behavior into one runtime.

## Why Consolidation Was Necessary

As the repo expanded, communication and task orchestration stopped being separable concerns.

Email, meeting artifacts, project tasks, and scheduled follow-up work all needed:

- the same persona routing model
- the same persistence layer
- the same policy boundaries
- the same audit trail

Splitting those responsibilities across multiple small daemons increased ambiguity without delivering useful isolation.

## What The Unified Runtime Owns Now

- inbound Microsoft Graph webhook handling
- OpenProject webhook ingestion
- fallback task polling
- scheduled automation entrypoints
- work-item persistence and correlation
- execution attempts and leases
- policy shaping for downstream side effects

## What Improved

### Consistent Execution Semantics

A unified runtime means the system can apply the same rules for deduplication, retries, and auditability regardless of whether work arrived from email, a project task, or a scheduled sweep.

### One Collaboration Model

OpenProject tasks, inbox work, and meeting follow-up no longer behave like separate products. They are different entrypoints into the same orchestration layer.

### Better Governance

The system now has a clearer place to enforce reply authority, approval state, and allowed side effects.

## What The Consolidation Did Not Do

It did not eliminate the distinction between collaborative project state and execution reliability state.

That split still matters:

- OpenProject remains the shared project board
- Postgres remains the execution ledger

The unified runtime coordinates those systems; it does not collapse them into one.
