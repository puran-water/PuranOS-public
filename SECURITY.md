# Security Policy

PuranOS is built around the assumption that operational automation is only useful if it is also governable.

## Reporting A Vulnerability

Please report vulnerabilities privately through GitHub Security Advisories or another direct private channel to the maintainers. Do not open public issues for security-sensitive findings.

## What This Repo Tries To Protect

The highest-risk surfaces in this monorepo are:

- OAuth and API credentials
- inbound webhook secrets
- persona-specific authority boundaries
- live infrastructure details
- agent side effects such as outbound mail or task mutation

## Tracked File Policy

Tracked files should be safe to publish.

That means:

- no live secrets
- no production tokens
- no private hostnames or identifying email addresses in docs and templates
- no workstation-specific local paths in public-facing documentation

Tracked `.example`, `.template`, and README files should act as documentation, not as a copy of a live environment.

## Runtime Secret Management

- Real `.env` files are untracked.
- Tokens and OAuth artifacts are stored outside the repository or in ignored runtime locations.
- Secret-bearing deployment values should be injected per environment, not committed.

## Agent Runtime Security Model

### Persona Scoping

Each persona has its own configuration and tool surface. That is the first boundary.

### Tool-Level Enforcement

For sensitive mutations, the runtime enforces policy where the tool executes. That matters because prompt-only restrictions are not sufficient for production automation.

### Durable Audit Trails

The communication and orchestration stack records execution attempts, state transitions, and side effects so actions can be reconstructed after the fact.

## Infrastructure Hygiene

- Prefer internal origin routing for automated service-to-service traffic where appropriate.
- Keep public ingress separate from internal control paths.
- Provision OpenProject custom fields and outgoing webhooks from repo-owned manifests.
- Review service templates before public release to ensure placeholders remain generic.

## Release Checklist

Before public release or open-sourcing selected parts of this repo:

1. grep documentation and templates for private domains, email addresses, IPs, and local paths
2. verify `.env` and token files remain ignored
3. confirm example configs use placeholders only
4. confirm architecture docs describe the current system rather than stale deployment history
