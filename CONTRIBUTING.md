# Contributing To PuranOS

PuranOS is an integration-heavy monorepo. Good contributions improve leverage, clarity, and operational reliability at the same time.

## Contribution Principles

Prioritize changes that make the system:

- easier to reason about
- safer to operate
- more reusable across personas
- more explicit about state and side effects

## What Belongs Where

### MCP Servers

Put integration logic, typed operations, and API adaptation in servers.

### Personas

Put role boundaries, escalation rules, and operating context in `agent-configs/`.

### Skills

Put reusable workflows and quality expectations in `skills/`.

### Orchestration

Put ingress reliability, retries, leases, and execution ledger behavior in long-running agents such as `communication-agent`.

## Documentation Expectations

If you change behavior, update the docs that explain that behavior.

This repo especially needs documentation updates when you change:

- server contracts
- persona boundaries
- deployment requirements
- data models
- workflow state semantics
- webhook or polling behavior

Keep documentation sanitized:

- use `example.com`
- use placeholder mailboxes
- use repo-relative paths or `/path/to/...`
- do not paste secrets or live host details

## Development Workflow

1. inspect the relevant server, agent, skill, or service before editing
2. keep changes scoped
3. update docs and templates when contracts change
4. verify with the narrowest meaningful test surface first
5. do not revert unrelated user work in a dirty tree

## Testing Guidance

This repo contains multiple independent codebases. Run the tests that match the area you changed.

Common patterns:

- Python servers: `uv run pytest`
- Python syntax checks: `python3 -m py_compile ...`
- Node services: `npm run build`, `npm run test`, or `node --check`
- server-specific smoke tests where available

For orchestration changes, include at least one validation of:

- ingress behavior
- mutation safety
- state persistence or audit trails

## Changes That Require Extra Care

### OpenProject Orchestration

Changes involving agent-state custom fields, webhook delivery, polling, or execution-ledger behavior can affect the entire collaboration model. Update the OpenProject and communication-agent docs when changing these paths.

### Shared Templates

Changes to `.example` and `.template` files should preserve the repo's public-sanitized posture.

### Skills and Persona Contracts

If a persona gains or loses tool access, the surrounding docs should say why.

## Pull Requests

Strong pull requests in this repo usually include:

- a clear scope
- a concrete reason for the change
- contract or migration notes when needed
- updated documentation
- proof of validation
