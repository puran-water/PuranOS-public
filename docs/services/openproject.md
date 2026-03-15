# OpenProject Upgrade Notes

This repo uses the self-hosted OSS BIM image track and treats OpenProject as a core collaboration system, not a disposable SaaS dependency. Upgrade decisions therefore have to protect both human-facing workflows and agent orchestration behavior.

## Current Repository Assumptions

- OpenProject is the long-running project board for collaborative work.
- Agent execution state is stored in Postgres, but agent task state is surfaced in OpenProject via typed custom fields.
- Native OpenProject outgoing webhooks are used when available.
- The communication-agent operates in webhook-preferred mode with a poller fallback for correctness.

## Image Policy

- keep `OPENPROJECT_IMAGE_TAG` explicit
- do not auto-jump major versions
- stage major upgrades one major at a time
- use the OSS BIM track without assuming an enterprise token

Current staged path documented in the repo:

1. `15.5.1-slim`
2. `16.6.8-slim`
3. `17.2.0-slim-bim` on `amd64`, or `17.2.0-slim` with `OPENPROJECT_EDITION=bim` on `arm64`

## Why Upgrades Need Care

OpenProject is not just another application in this stack. It participates directly in:

- project coordination
- agent routing
- typed task state
- webhook-triggered execution

An upgrade therefore has to protect:

- API behavior
- custom field semantics
- webhook delivery
- comment and journal behavior
- task lifecycle mutations

## Before Each Upgrade

1. back up the OpenProject database and data volumes
2. record the current image tag and environment values
3. confirm OpenProject custom fields used for agent state still exist and map correctly
4. confirm the communication-agent migration set is already applied
5. validate MCP connectivity against a non-production or low-risk project first

## Runtime Sizing Guidance

The current default sizing is tuned for agent-heavy API traffic with a small human team.

Tracked defaults in `.env.template`:

- `OPENPROJECT_WEB_MEM_LIMIT=3072m`
- `OPENPROJECT_WORKER_MEM_LIMIT=1536m`
- `OPENPROJECT_CRON_MEM_LIMIT=256m`
- `OPENPROJECT_CACHE_MEM_LIMIT=256m`
- `OPENPROJECT_RAILS_MAX_THREADS=8`
- `OP_WEB_DB_POOL=12`
- `OP_WORKER_DB_POOL=8`
- `OP_CRON_DB_POOL=4`

The objective is not raw concurrency at any cost. It is stable response behavior under automation load.

## Internal Agent Routing

For automated writes, prefer internal origin access instead of sending MCP or agent traffic through the public edge.

The repo includes a workstation helper:

- `oci-openproject-tunnel.service`

When using a raw tunnel or internal HTTP origin, clients may also need:

- `OPENPROJECT_HOST_HEADER`
- `OPENPROJECT_FORWARDED_PROTO`

so the application still sees its canonical host and protocol context.

## Agent-State Custom Fields

OpenProject custom fields for agent state should be provisioned from source control.

Tracked source of truth:

- `agent-custom-fields.yml`
- `scripts/ensure_agent_custom_fields.rb`

The provisioner is idempotent and prints the mapping needed for:

- `agents/communication-agent`
- `servers/openproject-mcp`

## Outgoing Webhook Provisioning

The OpenProject-to-communication-agent webhook should also be provisioned from source control.

Tracked source of truth:

- `agent-webhook.yml`
- `scripts/ensure_communication_agent_webhook.rb`

Default event set:

- `work_package:created`
- `work_package:updated`
- `work_package_comment:comment`
- `work_package_comment:internal_comment`

## Validation Checklist

After upgrade:

1. verify the UI loads correctly
2. create and update a work package manually
3. validate OpenProject API connectivity
4. create and complete an agent task through `openproject-mcp`
5. verify the communication-agent can receive webhooks, poll fallback work, post comments, and complete tasks
6. confirm typed custom fields remain readable and writable

## Watchtower Guidance

Watchtower can be useful for update visibility, but not for unattended OpenProject major upgrades. The right operating model here is:

- explicit version pinning
- staged upgrades
- deliberate validation

not “always latest” automation on a system that sits in the middle of human and agent collaboration.
