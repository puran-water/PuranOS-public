# Communication Agent Automation Architecture

This document describes the active automation model in the current codebase.

## Inbound Channels

The service currently accepts work from four main sources:

| Source | Purpose |
|--------|---------|
| Microsoft Graph email | mailbox-routed specialist work |
| Microsoft Graph calendar / transcripts | meeting-related workflows and follow-up processing |
| OpenProject outgoing webhooks | project-task collaboration and result updates |
| Scheduled jobs | recurring briefs and operational checks |

## Routing Model

### Email and Graph-Driven Work

Incoming work is correlated to the correct persona using mailbox identity, sender policy, and the configured mailbox map. The goal is deterministic routing rather than a broad classifier trying to guess organizational intent.

### OpenProject-Driven Work

OpenProject tasks are routed using a combination of:

- typed agent-state fields
- assignee or responsible-party resolution
- predecessor status
- approval state
- blocked reason

This keeps project collaboration visible in the board while still allowing the runtime to enforce execution rules.

## Execution Controls

### Deduplication

External deliveries are recorded before execution so duplicate webhook deliveries do not result in duplicate work.

### Attempts and Leases

Every execution path that matters operationally should have:

- a lease owner
- an attempt record
- transition history

That is the foundation for restart-safe behavior.

### Side-Effect Policy

Mutation policies are enforced at the tool boundary. For Office-oriented operations this means outbound mail, Planner, and Calendar writes can be blocked or allowed by runtime policy rather than prompt convention alone.

## OpenProject Workflow Semantics

### Webhook-Preferred

When OpenProject native webhooks are configured, the service treats them as the primary change detector.

### Poller-Fallback

The poller remains enabled because task eligibility can change without a fresh event on the task itself, especially when human-completed predecessor work unblocks a downstream agent task.

### Human And Agent Coexistence

The board is intentionally shared:

- humans can create or edit tasks directly
- agents can create, claim, comment on, and complete work packages
- typed agent-state fields keep machine state explicit without hiding it in narrative comments

## Persistence And Audit

The service records:

- ingress events
- work items
- attempts
- state transitions
- side effects
- conversation correlation

This is what makes the automation suitable for enterprise workflows rather than just one-shot assistant interactions.

## Operational Implication

The value of the automation layer is not just convenience. It is that it gives the organization one place where:

- inbound work becomes structured execution
- execution is durable and auditable
- project state stays legible to humans
