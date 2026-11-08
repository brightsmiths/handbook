# Brightsmiths handbook

System-level documentation and architecture for the Brightsmiths tools: a set of small services
and agent profiles that take engineering work from a GitHub issue to a published GitHub Release.

This repository contains no code. Each service's design and documentation live in that
service's own repository; the [service catalogue](docs/architecture/services.md) links to them.

## Contents

| Document | What it covers |
| --- | --- |
| [Overview](docs/overview.md) | Purpose, scope, guiding principles |
| [System architecture](docs/architecture/system.md) | Layers, components, event flow, shared infrastructure |
| [Service catalogue](docs/architecture/services.md) | Every service: purpose, role in the factory, required changes |
| [Agent profiles and discipline packs](docs/architecture/agent-profiles.md) | How CogWorks picks agents; trust zones; extending to other disciplines |
| [Workflow, gates and tiers](docs/architecture/workflow.md) | Stages, gates G1–G9, tiers T0–T3, human touchpoints |
| [Oracles and anti-faking](docs/architecture/oracles.md) | Keeping the oracle out of the author's reach |
| [State model](docs/architecture/state-model.md) | How GitHub carries all state |
| [Isolation and security](docs/architecture/isolation-and-security.md) | microVM sandboxes, identities, prompt-injection posture, provenance |
| [Operating the factory](docs/architecture/operations.md) | Health loops, observability, harness change management |
| [Roadmap](docs/roadmap.md) | Phased plan and per-service changes |
| [Open questions](docs/open-questions.md) | Decisions not yet made |
| [ADRs](docs/adr/) | System-level architecture decisions |
