# Communication Agent Documentation Index

This directory documents the current orchestration service rather than the earlier split-service architecture.

## Start Here

- [README.md](README.md) — what the service is, why it exists, and how it fits into the repo
- [AUTOMATION.md](AUTOMATION.md) — current inbound channels, routing model, and policy behavior
- [CONSOLIDATION.md](CONSOLIDATION.md) — how the service evolved into the current unified runtime

## When To Read Which File

### You want to understand the system

Read [README.md](README.md).

### You want to understand actual event handling and task dispatch

Read [AUTOMATION.md](AUTOMATION.md).

### You want to understand why the runtime is unified and why OpenProject is handled the way it is

Read [CONSOLIDATION.md](CONSOLIDATION.md).

## Related Runtime Files

- `src/main.py`
- `src/op_poller.py`
- `src/session_store.py`
- `src/policy.py`
- `src/specialist_caller.py`
- `src/webhook/`
- `src/subscriptions/`
