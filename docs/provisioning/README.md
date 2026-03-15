# Provisioning Guide

PuranOS provisioning is template-driven and sanitized by default. The deployment model is shaped by the needs of an industrial wastewater and waste-to-value design-build firm, but the patterns are intentionally reusable for other industrial engineering organizations with comparable operational complexity.

## Why This Exists

Provisioning docs are usually where sensitive details accumulate:

- public hostnames
- SSH targets
- email addresses
- token paths
- service account assumptions

This guide documents the deployment model while keeping those values out of tracked files.

## What The Repo Assumes Today

The repo currently targets a self-hosted single-host or small-cluster deployment model with:

- Docker Compose for core service stacks
- systemd for long-running agents and helper tunnels
- PostgreSQL for shared persistent state
- SSH tunnels or internal origin routing for workstation-local agent access when appropriate

## Repo-Managed Provisioning Assets

In the internal operational monorepo, the deployment model is expressed through tracked templates and provisioners. This public repo documents those artifacts rather than shipping live deployment material.

### Service Templates

- OpenProject environment template
- CRM environment template
- Inventory environment template
- ingress and tunnel template
- shared database environment template

### OpenProject Provisioners

- OpenProject agent custom-field manifest
- OpenProject outgoing webhook manifest
- custom-field provisioner
- outgoing-webhook provisioner

### Agent Runtime Templates

- communication-agent environment example
- OpenProject MCP environment example

## Recommended Rollout Sequence

1. Bootstrap host dependencies and clone the repo.
2. Render `.env` files from tracked templates with environment-specific values.
3. Start shared infrastructure and service stacks.
4. Apply database migrations for long-running agents.
5. Provision OpenProject typed custom fields and outgoing webhook definitions from the repo manifests.
6. Configure long-running agents and any workstation-local tunnel helpers.
7. Validate server connectivity and end-to-end task flows.

## Deployment Principles

### Prefer Direct Origin Access For Internal Automation

Where an internal agent is talking to a self-hosted service, prefer direct origin access over sending automated traffic through the public edge. That reduces latency and eliminates avoidable proxy failure modes.

### Keep Public and Internal Contracts Separate

Humans can use the public application URL. Agents and MCP servers often benefit from a different, internal route so they can avoid extra ingress layers while still honoring canonical host and protocol requirements.

### Provision Shared Metadata From Source Control

Typed agent-state fields and outgoing webhooks should be provisioned from repo-managed manifests, not clicked into existence manually. That is the only way to keep the collaboration model reproducible.

### Treat Shared Postgres As More Than App Storage

The shared database stack now also carries orchestration state for the communication-agent. That means deployment validation should cover not only application schemas, but also the execution-ledger database used for deduplication, attempts, transitions, and side-effect audit records.

## Release Hygiene

Before publishing docs or pushing public-facing branches:

- confirm examples use placeholder domains such as `example.com`
- confirm paths use placeholders or repo-relative references
- confirm no secrets or live tokens appear in tracked files
- confirm operational runbooks still match the current codebase

See [SECURITY.md](../../SECURITY.md).
