# ADR-0007: Discipline-neutral core; profiles and checks shipped as discipline packs

**Status:** Proposed
**Date:** 2026-11-08

## Context

The factory should be usable for engineering disciplines other than software, such as mechanical
or electrical design. CogWorks already delegates domain toolchains to external domain services
through its Extension API. Brightsmiths only intends to build software support.

## Decision

- CogWorks, gate-warden, sandbox-runner, agent-gate and chain-chronicler stay
  discipline-neutral.
- A discipline pack supplies: profiles per stage type, skills, stage graphs per tier, a domain
  service, oracle conventions, gate check commands, and path conventions.
- Brightsmiths maintains only the software pack (Rust first). Other packs can live in other
  repositories and are referenced by version from the harness.
- The invariants of ADR-0003 and ADR-0004 apply to every pack.

## Consequences

- The gate result format and the stage-contract schema become public interfaces that need
  versioning.
- Software-specific tools (oracle-guard, ratchet) are part of the software pack, not the core.

## Alternatives considered

- **Software only:** simpler, but forecloses an explicit goal.
- **A fully generic oracle abstraction up front:** premature. The interface is formalised after
  the software pack is proven.
